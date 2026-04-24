<p align="center">
  <h1 align="center">Web Smith</h1>
  <p align="center">
    <strong>Give it a URL. Get back Lighthouse 100.</strong>
  </p>
  <p align="center">
    <code>2 skills</code> &middot; <code>7 headless tools</code> &middot; <code>pixel-perfect diffs</code> &middot; <code>1680 lines of battle-tested process</code>
  </p>
</p>

<p align="center">
  <a href="https://astro.build"><img src="https://img.shields.io/badge/Astro_6-BC52EE?style=flat-square&logo=astro&logoColor=white" alt="Astro"></a>
  <a href="https://tailwindcss.com"><img src="https://img.shields.io/badge/Tailwind_v4-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white" alt="Tailwind v4"></a>
  <a href="https://pages.cloudflare.com"><img src="https://img.shields.io/badge/Cloudflare_Pages-F38020?style=flat-square&logo=cloudflare&logoColor=white" alt="Cloudflare Pages"></a>
  <a href="https://claude.ai"><img src="https://img.shields.io/badge/Claude_Code-CC785C?style=flat-square&logo=anthropic&logoColor=white" alt="Claude Code"></a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Performance-100-00C853?style=for-the-badge" alt="Performance 100">
  <img src="https://img.shields.io/badge/Accessibility-100-00C853?style=for-the-badge" alt="Accessibility 100">
  <img src="https://img.shields.io/badge/Best_Practices-100-00C853?style=for-the-badge" alt="Best Practices 100">
  <img src="https://img.shields.io/badge/SEO-100-00C853?style=for-the-badge" alt="SEO 100">
</p>

---

```
claude plugin marketplace add harryy2510/claude-toolkit
claude plugin install web-smith@claude-toolkit
```

---

<p align="center">
  <img src="docs/web-smith-flow.png" alt="Web Smith Flow" />
</p>

---

## The Problem

Marketing sites built on WordPress, HubSpot, Kajabi, Squarespace, and Wix are slow. They ship 2MB of JavaScript, load 14 render-blocking stylesheets, score 40 on Lighthouse, and cost $50-300/month to host.

Web Smith takes any marketing site and rebuilds it as a static site that scores 100/100/100/100, loads in under 1 second, costs $0/month to host, and adds dark mode + responsive even if the original had neither.

An AI agent runs the entire pipeline autonomously -- scrape, design, scaffold, build, optimize, audit, ship.

---

## Two Skills

| Skill | What | When |
|---|---|---|
| `website-cloner` | Full clone pipeline: scrape a live site, extract design tokens, scaffold Astro project, build pixel-perfect pages, invoke optimizer | "Clone snappykraken.com" |
| `site-optimizer` | Framework-agnostic optimization: images, fonts, SEO, AEO, accessibility, audit loop with 3 headless tools, visual pixel diff | "Optimize this site for Lighthouse 100" |

`website-cloner` runs phases 1-4 (scrape through build) then invokes `site-optimizer` for optimization, auditing, and shipping. `site-optimizer` works standalone on any static site.

---

## website-cloner

1226 lines. Covers every edge case from cookie banner removal to font licensing detection to i18n.

### Pipeline

| Phase | What happens | Tools |
|---|---|---|
| **1. Scrape** | Discover all pages from nav/footer links. Screenshot at 3 viewports (desktop/tablet/mobile) with overlay removal. Fetch rendered HTML. Download all assets via HAR. Extract design tokens. | `shot-scraper`, `dembrandt` |
| **2. Design System** | Map colors to semantic CSS variables. Derive dark mode via OKLCH lightness inversion. Build `/design-system` page with all tokens, typography, component states. | Manual + OKLCH math |
| **3. Scaffold** | Create Astro 6 project. Tailwind v4 CSS-based config. ESLint + Prettier + Husky. View Transitions. Theme toggle. Skip-to-content. Mobile menu. Full `.gitignore`, `_headers`, `_redirects`. | `bun create astro` |
| **4. Build Pages** | Content in `content.ts` (structured) or markdown via Content Collections (prose). All `.astro` components, zero JS. JSON-LD schemas per page type. Responsive always. i18n-ready. | Astro 6 |
| **5. Optimize** | Invokes `site-optimizer` skill | See below |

### What the skill provides (not just instructions)

Full, copy-paste-ready code for:

- `astro.config.ts` with sitemap, Tailwind vite plugin, path aliases
- `app.css` with complete theme system (CSS variables, scrollbar, selection, focus-visible, reduced-motion, view-transition resets)
- `base.astro` layout with `ClientRouter`, skip-to-content, named slots
- `base-head.astro` with all meta tags (OG, Twitter, theme-color with media queries, canonical, sitemap link, FOUC-prevention script)
- `theme-toggle.astro` with sun/moon icons, localStorage, system preference listener, `astro:after-swap` handler
- Mobile menu toggle with hamburger, `aria-expanded`, view transition cleanup
- `robots.txt.ts` API route
- `eslint.config.ts` (strict TS, perfectionist, inline type imports)
- `prettier.config.ts` (tabs, no semis, single quotes, astro plugin)
- `content.config.ts` with Zod schema + glob loader
- `[...slug].astro` blog route with `getStaticPaths` + `render`
- `404.astro` page
- `.gitignore` (comprehensive)
- Placeholder replacement table (7 placeholders, all documented)
- `lint-staged` config + `husky init` commands

### Edge cases handled

- Cookie consent banner removal (13 selectors: OneTrust, CookieBot, HubSpot, Intercom, GDPR, etc.)
- SPA detection (empty `<div id="root">` + rich screenshot = use `document.documentElement.outerHTML`)
- Font licensing detection (Adobe Fonts / Typekit, fonts.com, Hoefler flagged)
- Lazy-loaded content (scroll-to-bottom JS before screenshot capture)
- Anti-bot protection guidance (staging URL, IP whitelist)
- Third-party embeds (lite-youtube for YouTube, lazy iframes for maps/booking)
- Form backends for static sites (Formspree, Cloudflare Worker, mailto)
- Animations without JS (CSS `scroll-snap` carousels, `<details>` accordions, `IntersectionObserver`)
- i18n structure (single language default, multi-language with `hreflang` ready)
- Dark mode image treatment (brightness/contrast adjustment, SVG `currentColor`, logo variant swap, shadow adaptation)

### Hard rules

```
Zero JS by default          Only <script is:inline> for theme + mobile menu
Static output only          output: 'static', no SSR adapter
Dark mode always            Even if original has none -- derive via OKLCH
Responsive always           Even if original is not -- mobile-first
Self-hosted fonts           Never Google Fonts CDN -- subset + WOFF2
Semantic HTML               <nav>, <main>, <article>, <section>, skip-to-content
Content separated           content.ts or markdown, never hardcoded
Lighthouse 100 target       99+ acceptable after 200 audit attempts
```

---

## site-optimizer

454 lines. Framework-agnostic -- works on Astro, Next.js, Hugo, any static site. Image code examples use Astro syntax but all audit tools and checklists apply universally.

### Coverage

| Area | Details |
|---|---|
| **Images** | Astro `<Image>` with `width`/`height`/`alt`/`loading`. Responsive `widths`/`sizes` for hero. `fetchpriority="high"` on LCP only. `import.meta.glob` for dynamic collections with fallback. |
| **Fonts** | `pyftsubset` to Latin WOFF2. Variable fonts for 3+ weights. `font-display: swap`. Preload critical font with `crossorigin`. `@font-face` template. Never Google Fonts CDN. |
| **SEO** | Per-page checklist: title, description, author, keywords, canonical, Open Graph (8 tags), Twitter (4 tags), theme-color with media queries, sitemap link. JSON-LD catalog: Organization, WebSite, FAQPage, BreadcrumbList, Article, LocalBusiness, SoftwareApplication, Person. |
| **Accessibility** | Skip-to-content link (exact Tailwind classes). `focus-visible` outline. `aria-current="page"` on nav. Color contrast AAA (7:1). Touch targets 44px. `prefers-reduced-motion` with view-transition reset. |
| **Performance CSS** | Font smoothing on `html`. `text-rendering: optimizeLegibility` on `body`. `-webkit-tap-highlight-color: transparent`. Scrollbar styling (webkit + standards). `::selection` colors. |
| **Security headers** | Platform-agnostic reference table (Cloudflare Pages, Netlify, Vercel, Nginx, Apache). X-Frame-Options, HSTS, CSP, Referrer-Policy, Permissions-Policy. |

### Audit tools

All headless. No browser UI. Safe for autonomous agents and CI.

| Tool | What it checks | Install |
|---|---|---|
| `unlighthouse-ci` | Lighthouse scores per page with budget thresholds | `bun add -D unlighthouse` |
| `seomator` | 251 SEO rules, AEO/GEO readiness, structured data, AI bot access | `bun add -g @seomator/seo-audit` |
| `pa11y` | WCAG accessibility via axe-core (WCAG2AA/AAA) | `bun add -g pa11y` |
| `odiff` | Pixel-level image diff with percentage score | `bun add -g odiff-bin` |
| `magick compare` | Color extraction at exact pixel coordinates, SSIM | `brew install imagemagick` |
| `shot-scraper` | Element-level screenshots (`-s "nav"`, `-s "footer"`) | `pip install shot-scraper` |

### Audit workflow

```bash
bun run build && bun run preview &
sleep 2

bunx unlighthouse-ci                    # Lighthouse per page
seomator audit http://localhost:PORT \
  --crawl -m 50 --format json \
  -o .audit/seo-report.json              # 251 SEO rules + AEO

pa11y http://localhost:PORT \
  --runner axe --standard WCAG2AA \
  --chromium-arg=--no-sandbox \
  -r json > .audit/a11y-home.json        # WCAG per page

kill %1
```

Target: 100/100/100/100 + zero SEO errors + zero WCAG violations. Accept 99+ after 200 attempts.

### Visual comparison (the hard part)

AI agents build "close enough" not "pixel perfect." This is the fix:

```bash
# Element-level screenshots
shot-scraper original.com -s "nav" -o compare/original-nav.png
shot-scraper localhost:4321 -s "nav" -o compare/clone-nav.png

# Pixel diff
odiff compare/original-nav.png compare/clone-nav.png compare/diff-nav.png --parsable-stdout
# Output: "2400;2.00"  → 2% diff, needs fixing

# Extract exact color at coordinate
magick compare/original-nav.png -crop 1x1+200+50 txt:
# → 0,0: (27,29,33,255) #1B1D21FF

magick compare/clone-nav.png -crop 1x1+200+50 txt:
# → 0,0: (42,40,36,255) #2A2824FF  ← wrong, fix --ink token
```

Per-section diffs (nav, hero, footer, full page). Continue fixing until all sections < 1% diff. Fonts will cause differences -- focus on layout, spacing, colors when fonts differ.

---

## Prerequisites

```bash
# Scraping + design extraction
pip install shot-scraper && shot-scraper install
bun add -g dembrandt

# Auditing + visual diff
bun add -g odiff-bin @seomator/seo-audit pa11y
brew install imagemagick

# Excalidraw diagrams (optional)
bun add -g @swiftlysingh/excalidraw-cli

# Per-project (added during scaffold)
bun add -D unlighthouse
```

---

## Project Structure

```
web-smith/
├── plugins/web-smith/
│   ├── .claude-plugin/plugin.json
│   └── skills/
│       ├── website-cloner/
│       │   └── SKILL.md             1226 lines
│       └── site-optimizer/
│           └── SKILL.md              454 lines
├── docs/
│   ├── web-smith-flow.excalidraw    editable diagram (excalidraw.com)
│   └── web-smith-flow.png           rendered diagram
└── README.md
```

---

## Install

```bash
# Add marketplace (one time)
claude plugin marketplace add harryy2510/claude-toolkit

# Install web-smith
claude plugin install web-smith@claude-toolkit
```

Then in Claude Code:

```
You: "Clone snappykraken.com"
You: "Optimize this Astro site for Lighthouse 100"
```

---

## Part of claude-toolkit

| Plugin | What |
|---|---|
| [dotclaude](https://github.com/harryy2510/dotclaude) | 17 skills, 19 agents, 6 commands -- fullstack conventions |
| [vibe-pilot](https://github.com/harryy2510/vibe-pilot) | Kanban autopilot -- classify, triage, implement, report |
| **web-smith** | Website cloning + static site optimization |

```bash
claude plugin marketplace add harryy2510/claude-toolkit
claude plugin install dotclaude@claude-toolkit
claude plugin install vibe-pilot@claude-toolkit
claude plugin install web-smith@claude-toolkit
```

---

## Author

**Hariom Sharma** -- [github.com/harryy2510](https://github.com/harryy2510)

## License

MIT
