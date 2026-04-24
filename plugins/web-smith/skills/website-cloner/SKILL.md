---
name: website-cloner
description: "Use when cloning a customer's marketing website into a high-performance Astro static site. Triggers: clone site, replicate website, rebuild marketing site, website migration, site redesign for performance/SEO/accessibility, Lighthouse 100 rebuild. Covers full pipeline: scraping, design token extraction, Astro scaffold, component build, font/image optimization, dark mode, responsive, audit loop, Cloudflare Pages deploy."
user_invocable: true
---

# Website Cloner

Clone any marketing website into a pixel-perfect, high-performance Astro static site. Lighthouse 100. Cloudflare Pages. Dark mode. Responsive. Zero compromise.

## When to Use

- Customer hands you a URL and says "rebuild this but faster"
- Marketing site migration from WordPress/HubSpot/Kajabi/Squarespace/Wix
- Agency pitch: show their own site scoring 100/100/100/100

## Stack

| Layer | Tool |
|---|---|
| Framework | Astro 6+ (`output: 'static'`) |
| Styling | Tailwind CSS v4 (CSS-based config) |
| Deploy | Cloudflare Pages |
| Package manager | `bun` always, `bunx` not `npx` |
| Scraping | `shot-scraper` + `dembrandt` |
| Auditing | `unlighthouse-ci` (headless) |

## Prerequisites

```bash
pip install shot-scraper && shot-scraper install
bun add -g dembrandt odiff-bin @seomator/seo-audit pa11y
# unlighthouse installed per-project as devDep
# imagemagick via: brew install imagemagick
```

---

## Phase Overview

```
1. Scrape -> 2. Design System -> 3. Scaffold -> 4. Build Pages -> 5. Optimize + Audit + Ship (site-optimizer skill)
                                                                          |
                                                                     fix & repeat
```

---

## Phase 1: Scrape

### Pre-scrape checks

1. Verify `robots.txt` allows scraping (should be fine -- it's the customer's own site)
2. Check for anti-bot protection (Cloudflare Under Attack, DataDome). If blocked: ask customer for staging URL or temporary whitelist
3. Check for auth walls. If gated: use `shot-scraper auth URL` to capture login session, then pass `--auth auth.json` to all commands

### 1A. Discover pages

```bash
shot-scraper javascript "https://TARGET.com" "(() => {
  const links = new Set();
  document.querySelectorAll('nav a, footer a, header a, [role=navigation] a').forEach(a => {
    const url = new URL(a.href, location.origin);
    if (url.origin === location.origin && !url.hash) {
      links.add(url.pathname);
    }
  });
  return [...links].sort();
})()"
```

Save as `scrape/pages.json`. Also check:
- `sitemap.xml` for additional pages
- Subdomain links (e.g., `blog.target.com`) -- ask customer if in scope
- SPA detection: if homepage HTML body has `<div id="root">` with minimal content but screenshot shows rich content, it's an SPA -- use `shot-scraper javascript URL "document.documentElement.outerHTML"` instead of `shot-scraper html`

### 1B. Screenshot each page (3 viewports)

Create `screenshots/sites.yml`:

```yaml
- url: https://TARGET.com/
  output: screenshots/home-desktop.png
  width: 1280
  full_page: true
  wait: 3000
  javascript: |
    // Scroll to load lazy content
    await new Promise(r => {
      let h = 0;
      const i = setInterval(() => {
        window.scrollBy(0, 500);
        if (window.scrollY >= document.body.scrollHeight - window.innerHeight || h++ > 40) {
          clearInterval(i); window.scrollTo(0, 0); r();
        }
      }, 200);
    });
    // Remove overlays
    document.querySelectorAll('[class*=cookie], [class*=consent], [class*=modal], [id*=hubspot], [id*=hs-overlay], [class*=chat], .grecaptcha-badge, [class*=gdpr], [class*=onetrust], #CybotCookiebotDialog, .modal-backdrop, [id*=intercom], .leadinModal').forEach(el => el.remove());
    document.body.style.overflow = '';
    document.body.style.paddingRight = '';

- url: https://TARGET.com/
  output: screenshots/home-tablet.png
  width: 768
  full_page: true
  wait: 3000
  javascript: |
    document.querySelectorAll('[class*=cookie], [class*=consent], [class*=modal], [id*=hubspot], [id*=hs-overlay], [class*=chat], .grecaptcha-badge, [class*=gdpr], [class*=onetrust], #CybotCookiebotDialog, .modal-backdrop, [id*=intercom], .leadinModal').forEach(el => el.remove());
    document.body.style.overflow = '';

- url: https://TARGET.com/
  output: screenshots/home-mobile.png
  width: 375
  full_page: true
  wait: 3000
  javascript: |
    document.querySelectorAll('[class*=cookie], [class*=consent], [class*=modal], [id*=hubspot], [id*=hs-overlay], [class*=chat], .grecaptcha-badge, [class*=gdpr], [class*=onetrust], #CybotCookiebotDialog, .modal-backdrop, [id*=intercom], .leadinModal').forEach(el => el.remove());
    document.body.style.overflow = '';
```

Run: `shot-scraper multi screenshots/sites.yml --retina`

**For large sites (20+ pages):** script the YAML generation from `pages.json`.

### 1C. Fetch HTML source per page

```bash
mkdir -p scrape/html
# Use same overlay removal JS for HTML fetch
shot-scraper html "https://TARGET.com/" --javascript "document.querySelectorAll('[class*=cookie],[class*=consent],[class*=modal],[id*=hubspot],[id*=hs-overlay],[class*=chat],.grecaptcha-badge,[class*=gdpr],[class*=onetrust],#CybotCookiebotDialog,.modal-backdrop,[id*=intercom],.leadinModal').forEach(el=>el.remove());document.body.style.overflow=''" -o scrape/html/home.html
```

For SPAs: `shot-scraper javascript URL "document.documentElement.outerHTML" > scrape/html/home.html`

### 1D. Download assets (HAR extract)

```bash
shot-scraper har "https://TARGET.com/" -x -o scrape/assets
```

### 1E. Extract design tokens

```bash
dembrandt "https://TARGET.com" --json-only > scrape/design-tokens.json
dembrandt "https://TARGET.com" --dark-mode --json-only > scrape/design-tokens-dark.json 2>/dev/null || true
dembrandt "https://TARGET.com" --design-md > scrape/DESIGN.md
```

### 1F. Detect font licensing

Check scraped assets for restricted font services:
- `use.typekit.net` = Adobe Fonts (requires subscription)
- `fast.fonts.net` = Fonts.com
- `cloud.typography.com` = Hoefler&Co

If found: customer must provide licensed font files OR approve a visually similar open-source substitute.

### 1G. Build site metadata file

Create `scrape/site-data.json` with: domain, siteName, platform, pages, branding (logos, colors, fonts, darkMode), navigation, footer, seo, contact, forms, embeds, redirects.

### 1H. Catalog forms, embeds, and third-party scripts

During HTML review, identify:
- **Forms**: contact, newsletter, demo request. Document their `action` URLs and fields
- **Embeds**: YouTube, Vimeo, Google Maps, Calendly, Typeform. Document iframe sources
- **Third-party scripts**: analytics, chat widgets, tracking pixels. Document script sources
- **Video backgrounds**: note poster frames needed

---

## Phase 2: Design System

### 2A. Color palette

Map scraped colors to semantic CSS variables. ALWAYS create both light and dark mode even if original has no dark mode.

**Semantic token names (standard):**

| Token | Purpose |
|---|---|
| `--page` | Main background |
| `--page-2` | Secondary/alt background |
| `--page-3` | Tertiary (optional) |
| `--ink` | Primary text |
| `--ink-2` | Secondary text |
| `--muted` | Tertiary/disabled text |
| `--line` | Borders, dividers |
| `--line-soft` | Subtle separators |
| `--accent` | Primary brand color |
| `--accent-dark` | Hover/active state |
| `--accent-light` | Tinted background |

Additional as `--brand-*`, `--accent-2`, `--accent-3`.

**Dark mode color derivation (when original has no dark mode):**
1. Convert to OKLCH
2. Backgrounds: invert lightness (L: 0.97 -> 0.12, 0.95 -> 0.15)
3. Text: invert lightness (L: 0.15 -> 0.93, 0.35 -> 0.78)
4. Accents: increase lightness +5-10% for vibrancy on dark backgrounds
5. Borders: keep hue/chroma, invert lightness, reduce opacity
6. Accent-light tints: make them dark tints
7. Verify contrast ratios remain AAA (7:1+)

### 2B. Typography

Map font families to Tailwind `@theme` variables:

```css
@theme {
  --font-display: 'Manrope', system-ui, sans-serif;
  --font-sans: 'Source Sans Pro', system-ui, sans-serif;
  --font-mono: ui-monospace, 'SF Mono', monospace;
}
```

### 2C. Build `/design-system` page

Create `src/pages/design-system.astro` showing: color palette swatches (all tokens, light + dark), typography scale (h1-h6, body, small), spacing, component states, button/card variants, form elements, shadows, logo variants.

This page is the QA reference. Keep it the entire project life.

### 2D. Dark mode images

- Photos: add `class="dark:brightness-90 dark:contrast-105"` to slightly reduce brightness
- Screenshots with white backgrounds: add `class="rounded-lg dark:ring-1 dark:ring-[var(--line)]"`
- SVGs with hardcoded colors: replace with `currentColor` or provide light/dark variants
- Logo dark variants: use `class="dark:hidden"` / `class="hidden dark:block"` pattern
- Box shadows: `shadow-lg dark:shadow-none dark:ring-1 dark:ring-[var(--line)]` or use stronger opacity in dark

---

## Phase 3: Scaffold Project

### 3A. Create project

```bash
bun create astro@latest PROJECT_NAME -- --template minimal --no-install
cd PROJECT_NAME
```

### 3B. Install dependencies

```bash
bun add astro @astrojs/sitemap clsx tailwind-merge
bun add -D @tailwindcss/vite tailwindcss typescript @astrojs/check eslint @eslint/js typescript-eslint eslint-plugin-perfectionist prettier prettier-plugin-astro husky is-ci lint-staged unlighthouse @types/node
```

### 3C. Project structure

```
src/
├── assets/images/          # ALL raster images here (Astro optimizes)
│   ├── heroes/
│   ├── logos/
│   └── photos/
├── components/
│   ├── layout/             # base-head, nav, footer, theme-toggle
│   ├── sections/           # Reusable section components
│   └── ui/                 # Atomic: buttons, cards, badges
├── content/                # Markdown content (if blog/case studies)
│   └── blog/
├── content.config.ts       # Astro Content Collections (if using markdown)
├── data/
│   └── content.ts          # Structured content (nav, footer, sections)
├── layouts/
│   └── base.astro          # Root layout
├── libs/
│   └── cn.ts               # clsx + tailwind-merge
├── pages/
│   ├── index.astro
│   ├── design-system.astro
│   ├── 404.astro           # Custom 404 page
│   ├── robots.txt.ts
│   └── [other-pages].astro
└── styles/
    ├── app.css             # Tailwind + @theme + CSS variables
    └── fonts.css           # @font-face (if custom fonts)
public/
├── fonts/                  # Self-hosted .woff2 files
├── images/logos/           # SVGs served as-is
├── favicon.svg
├── favicon.ico
├── apple-touch-icon.png
├── og-image.png
├── _redirects              # Cloudflare Pages redirects
└── _headers                # Cloudflare Pages security headers
scrape/                     # Scraping artifacts (gitignored)
screenshots/                # Visual reference (committed)
└── sites.yml
unlighthouse.config.ts
CLONE-WIKI.md               # Evergreen project wiki
```

### 3D. Config files

**Read `references/scaffold-configs.md`** for all config file templates: astro.config.ts, tsconfig.json, package.json, eslint, prettier, .gitignore, app.css, cn.ts, base.astro, base-head.astro, theme-toggle.astro, robots.txt.ts, _headers, _redirects.

---

## Phase 4: Build Pages

### Page build order

1. Base layout + nav + footer + theme toggle
2. Design system page
3. Homepage (most sections)
4. Secondary pages
5. Blog/case study templates (if applicable)
6. 404 page
7. Robots.txt, sitemap (auto-generated)

### Content and component patterns

**Read `references/component-patterns.md`** for: content strategy (content.ts vs Content Collections), component rules, mobile menu toggle, active nav link, CSS-only dropdown, logo variants, embeds, forms, 404, animations/carousels, and i18n patterns.

### SEO and structured data

Handled by the `site-optimizer` skill (SEO checklist + JSON-LD). All SEO meta tags are already wired into the `base-head.astro` template from `references/scaffold-configs.md`.

---

## Phase 5: Optimize, Audit, and Ship

**REQUIRED:** Invoke the `site-optimizer` skill for all optimization, auditing, and shipping.

### Clone-specific additions on top of `site-optimizer`:

After running the optimizer, also verify:
- [ ] All pages visually match original (use `site-optimizer` visual comparison)
- [ ] Design system page complete with all tokens
- [ ] Dark mode derived correctly (even if original has no dark mode)
- [ ] Dark mode images: photos dimmed, SVGs using currentColor, logo variants swapping
- [ ] View Transitions working (`ClientRouter` + `astro:after-swap`)
- [ ] Forms functional or gracefully handled
- [ ] No dead links (all nav/footer links resolve)

### CLONE-WIKI.md

Create/update after every session: decisions and rationale, issues resolved, current audit scores per page, remaining work, font licensing notes, dark mode derivation notes, deviations from original and why, customer feedback log.

---

## Hard Rules

| Rule | Details |
|---|---|
| Zero JS by default | Only inline `<script is:inline>` for theme + mobile menu. No frameworks. |
| Static only | `output: 'static'`. No SSR adapter. |
| All images optimized | `<Image>` from `astro:assets` with width/height/alt/loading. In `src/assets/`, never `public/`. |
| Responsive always | Even if original is not. Mobile-first. |
| Dark mode always | Even if original has none. Derive colors. |
| Lighthouse 100 | Target 100/100/100/100. 99+ after 200 attempts. |
| Self-hosted fonts | Never Google Fonts CDN. Subset + self-host. |
| Semantic HTML | `<nav>`, `<main>`, `<article>`, `<section>`. Skip-to-content link. |
| Tailwind only | No CSS modules, styled-components, inline styles. |
| Content separated | `content.ts` or markdown. Never hardcoded in components. |
| `bun` always | Never npm/yarn. |
| Unique port | 4300-4399. |
| `as const` content | Full literal type narrowing. |
| `CLONE-WIKI.md` | Updated every session. |
| Optimize with `site-optimizer` | Always invoke `site-optimizer` skill for audit loop + ship. |

---

## Quick Reference

| Task | Command |
|---|---|
| Discover pages | `shot-scraper javascript URL "(() => { ... })()"` |
| Screenshot pages | `shot-scraper multi screenshots/sites.yml --retina` |
| Fetch HTML | `shot-scraper html URL -o file.html` |
| Download assets | `shot-scraper har URL -x -o dir` |
| Extract tokens | `dembrandt URL --json-only > tokens.json` |
| Dev server | `bun run dev` |
| Build | `bun run build` |
| Optimize + audit + ship | Invoke `site-optimizer` skill |
