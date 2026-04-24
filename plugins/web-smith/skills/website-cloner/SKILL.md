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

Create `scrape/site-data.json`:

```json
{
  "domain": "https://www.target.com",
  "siteName": "Target Brand",
  "platform": "HubSpot CMS",
  "pages": ["/", "/about", "/pricing", "/blog"],
  "branding": {
    "logos": {
      "primary": "path/to/logo.svg",
      "primaryDark": "path/to/logo-white.svg",
      "footer": "path/to/logo-footer.svg",
      "favicon": "path/to/favicon.svg"
    },
    "colors": {
      "primary": "#hex",
      "secondary": "#hex",
      "accent": "#hex",
      "background": "#hex",
      "backgroundAlt": "#hex",
      "text": "#hex",
      "textSecondary": "#hex",
      "textMuted": "#hex",
      "border": "rgba(...)",
      "success": "#hex",
      "error": "#hex"
    },
    "fonts": {
      "display": { "family": "Manrope", "weights": [600, 700, 800], "source": "google", "licensed": false },
      "body": { "family": "Source Sans Pro", "weights": [400, 600, 700], "source": "google", "licensed": false }
    },
    "darkMode": {
      "exists": false,
      "colors": {}
    }
  },
  "navigation": {
    "items": [
      { "label": "Products", "href": "/products", "children": [] }
    ]
  },
  "footer": {
    "columns": [
      { "heading": "Product", "links": [{ "label": "Features", "href": "/features" }] }
    ],
    "legal": "",
    "social": []
  },
  "seo": {
    "title": "Homepage title",
    "description": "Meta description",
    "ogImage": "path/to/og.png",
    "keywords": "",
    "googleVerification": ""
  },
  "contact": { "email": "", "phone": "", "address": "" },
  "forms": [],
  "embeds": [],
  "redirects": []
}
```

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

Create `src/pages/design-system.astro` showing:
- Color palette swatches (all tokens, light + dark side by side)
- Typography scale (h1-h6, body, small, with actual fonts rendering)
- Spacing samples
- Component states: hover, focus, active, visited
- Button variants
- Card variants
- Form elements
- Shadow examples (light vs dark mode)
- Logo variants (light/dark)

This page is the QA reference. Keep it the entire project life.

### 2D. Dark mode images

- Photos: add `class="dark:brightness-90 dark:contrast-105"` to slightly reduce brightness
- Screenshots with white backgrounds: add `class="rounded-lg dark:ring-1 dark:ring-[var(--line)]"`
- SVGs with hardcoded colors: replace with `currentColor` or provide light/dark variants
- Logo dark variants: use `class="dark:hidden"` / `class="hidden dark:block"` pattern
- Box shadows: `shadow-lg dark:shadow-none dark:ring-1 dark:ring-[var(--line)]` or use stronger opacity in dark

---

## Phase 3: Scaffold Project

### Placeholders to replace

All config templates use these placeholders. Replace ALL before running:

| Placeholder | Replace with | Example |
|---|---|---|
| `TARGET.com` | Customer's domain | `snappykraken.com` |
| `PROJECT_NAME` | Package/directory name | `snappykraken-astro` |
| `UNIQUE_PORT` | Unique port 4300-4399 | `4321` |
| `PROJ-theme` | Project-prefixed localStorage key | `sk-theme` |
| `FontName` | Actual font family name | `Manrope` |
| `PAGE_BG_LIGHT` | Light mode `--page` hex value | `#ffffff` |
| `PAGE_BG_DARK` | Dark mode `--page` hex value | `#121214` |

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

**`astro.config.ts`:**
```typescript
import sitemap from '@astrojs/sitemap'
import tailwindcss from '@tailwindcss/vite'
import { defineConfig } from 'astro/config'

export default defineConfig({
  integrations: [sitemap()],
  output: 'static',
  site: 'https://TARGET.com',
  server: { port: UNIQUE_PORT },
  vite: {
    plugins: [tailwindcss()],
    resolve: { alias: { '@': '/src' } },
  },
})
```

Port: 4300-4399 range. Check existing projects for conflicts.

If adding analytics: add `@astrojs/partytown` with `forward` config for the specific analytics SDK.

**`tsconfig.json`:**
```json
{
  "extends": "astro/tsconfigs/strict",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": { "@/*": ["src/*"] }
  }
}
```

**`package.json` additions:**
```json
{
  "overrides": { "vite": "7.3.2" },
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro preview",
    "check": "astro check && eslint . && prettier --check .",
    "fix": "eslint --fix . && prettier --write .",
    "prepare": "is-ci || husky"
  },
  "lint-staged": {
    "*.{ts,tsx,astro}": ["eslint --fix --cache", "prettier --write"],
    "*.{json,css,md,jsonc,toml}": ["prettier --write"]
  }
}
```

**Husky setup:**
```bash
bunx husky init
echo "bunx lint-staged --allow-empty" > .husky/pre-commit
echo "bun check" >> .husky/pre-commit
```

**`eslint.config.ts`:**
```typescript
import js from '@eslint/js'
import perfectionist from 'eslint-plugin-perfectionist'
import { defineConfig, globalIgnores } from 'eslint/config'
import tseslint from 'typescript-eslint'

export default defineConfig([
  globalIgnores(['.astro/', 'dist/', '.unlighthouse/']),
  js.configs.recommended,
  tseslint.configs.strict,
  perfectionist.configs['recommended-natural'],
  {
    rules: {
      '@typescript-eslint/consistent-type-imports': ['error', { fixStyle: 'inline-type-imports' }],
      '@typescript-eslint/no-empty-object-type': 'error',
      eqeqeq: 'error',
    },
  },
])
```

**`prettier.config.ts`:**
```typescript
import type { Config } from 'prettier'

const config: Config = {
  arrowParens: 'always',
  plugins: ['prettier-plugin-astro'],
  semi: false,
  singleQuote: true,
  tabWidth: 2,
  trailingComma: 'all',
  useTabs: true,
  overrides: [{ files: ['*.jsonc'], options: { parser: 'json' } }],
}

export default config
```

**`.gitignore`:**
```
# Build
dist/
.astro/
*.tsbuildinfo

# Dependencies
node_modules/

# Environment
.env
.env.*
!.env.example
!.env.*.encrypted

# IDE
.idea/
.vscode/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Tooling
.wrangler/
.eslintcache
.cache
.unlighthouse/
.audit/
.remember
.superpowers/
coverage/
*.lcov
logs/
*.log
*.tgz

# Scraping artifacts (reference only, not deployed)
scrape/
```

**`src/styles/app.css`** (complete):
```css
@import 'tailwindcss';
@import './fonts.css'; /* Remove this line if using system fonts */

@custom-variant dark (&:where(.dark, .dark *));

@theme {
  --font-display: 'FontName', system-ui, sans-serif;
  --font-sans: 'FontName', system-ui, sans-serif;
  --font-mono: ui-monospace, 'SF Mono', 'Cascadia Code', monospace;
}

@theme inline {
  --color-page: var(--page);
  --color-page-2: var(--page-2);
  --color-ink: var(--ink);
  --color-ink-2: var(--ink-2);
  --color-muted: var(--muted);
  --color-line: var(--line);
  --color-line-soft: var(--line-soft);
  --color-accent: var(--accent);
  --color-accent-dark: var(--accent-dark);
  --color-accent-light: var(--accent-light);
}

:root {
  --page: #ffffff;
  --page-2: #f8f9fb;
  --ink: #1b1d21;
  --ink-2: #3d4049;
  --muted: #6b7280;
  --line: rgba(27, 29, 33, 0.1);
  --line-soft: rgba(27, 29, 33, 0.05);
  --accent: #d43a38;
  --accent-dark: #bf2e2c;
  --accent-light: #fef2f2;
}

html.dark,
.dark {
  --page: #121214;
  --page-2: #1c1c1f;
  --ink: #f0eeeb;
  --ink-2: #c9c5be;
  --muted: #8a8680;
  --line: rgba(240, 238, 235, 0.12);
  --line-soft: rgba(240, 238, 235, 0.06);
  --accent: #e85450;
  --accent-dark: #f06e6a;
  --accent-light: #2a1515;
}

html {
  scroll-behavior: smooth;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  scrollbar-color: color-mix(in oklab, var(--line) 80%, transparent) transparent;
  scrollbar-width: thin;
}

body {
  background: var(--page);
  color: var(--ink);
  font-family: var(--font-sans);
  font-size: 16px;
  line-height: 1.6;
  -webkit-tap-highlight-color: transparent;
  text-rendering: optimizeLegibility;
}

::selection {
  background: var(--accent);
  color: #fff;
}

:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}

::-webkit-scrollbar { width: 6px; }
::-webkit-scrollbar-track { background: transparent; }
::-webkit-scrollbar-thumb {
  background: color-mix(in oklab, var(--line) 80%, transparent);
  border-radius: 3px;
}

@media (prefers-reduced-motion: reduce) {
  html { scroll-behavior: auto; }
  ::view-transition-group(*),
  ::view-transition-old(*),
  ::view-transition-new(*) {
    animation-duration: 0s !important;
  }
}
```

**`src/libs/cn.ts`:**
```typescript
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export const cn = (...inputs: ClassValue[]) => twMerge(clsx(inputs))
```

**`src/layouts/base.astro`:**
```astro
---
import BaseHead from '@/components/layout/base-head.astro'
import Footer from '@/components/layout/footer.astro'
import Nav from '@/components/layout/nav.astro'
import '@/styles/app.css'
import { ClientRouter } from 'astro:transitions'

type Props = {
  description?: string
  title?: string
}

const headProps = Astro.props
---

<!doctype html>
<html lang="en">
  <head>
    <BaseHead {...headProps}>
      <slot name="head" />
    </BaseHead>
    <ClientRouter />
  </head>
  <body class="bg-page text-ink antialiased">
    <a
      class="sr-only focus:not-sr-only focus:fixed focus:left-4 focus:top-4 focus:z-50 focus:rounded-full focus:bg-ink focus:px-4 focus:py-2 focus:text-page focus:outline-none"
      href="#main-content"
    >
      Skip to content
    </a>
    <Nav />
    <main id="main-content">
      <slot />
    </main>
    <Footer />
    <slot name="after-footer" />
  </body>
</html>
```

**`src/components/layout/base-head.astro`:**
```astro
---
import { content } from '@/data/content'

type Props = {
  description?: string
  title?: string
}

const siteUrl = content.meta.url
const currentPath = new URL(Astro.url.pathname, siteUrl).href
const { description = content.meta.description, title = content.meta.title } = Astro.props
---

<meta charset="utf-8" />
<meta content="width=device-width, initial-scale=1" name="viewport" />
<meta content="light dark" name="color-scheme" />

<title>{title}</title>
<meta content={description} name="description" />
<meta content={content.meta.author} name="author" />
<meta content={content.meta.keywords} name="keywords" />
<meta content="index, follow" name="robots" />

<link href={currentPath} rel="canonical" />
<link href="/favicon.svg" rel="icon" type="image/svg+xml" />
<link href="/favicon.ico" rel="icon" sizes="32x32" type="image/x-icon" />
<link href="/apple-touch-icon.png" rel="apple-touch-icon" sizes="180x180" />
<link href="/sitemap-index.xml" rel="sitemap" type="application/xml" />

<meta content="PAGE_BG_LIGHT" media="(prefers-color-scheme: light)" name="theme-color" />
<meta content="PAGE_BG_DARK" media="(prefers-color-scheme: dark)" name="theme-color" />

<!-- Open Graph -->
<meta content={currentPath} property="og:url" />
<meta content="website" property="og:type" />
<meta content={title} property="og:title" />
<meta content={description} property="og:description" />
<meta content={content.meta.siteName} property="og:site_name" />
<meta content="en_US" property="og:locale" />
<meta content={`${siteUrl}/og-image.png`} property="og:image" />
<meta content="1200" property="og:image:width" />
<meta content="630" property="og:image:height" />

<!-- Twitter -->
<meta content="summary_large_image" name="twitter:card" />
<meta content={title} name="twitter:title" />
<meta content={description} name="twitter:description" />
<meta content={`${siteUrl}/og-image.png`} name="twitter:image" />

<!-- Blocking theme script -->
<script is:inline>
;(function () {
  var t = localStorage.getItem('PROJ-theme')
  if (t === 'dark' || (!t && matchMedia('(prefers-color-scheme:dark)').matches)) {
    document.documentElement.classList.add('dark')
  }
})()
</script>

<slot />
```

Replace `PROJ-theme` with a project-specific localStorage key (e.g., `sk-theme` for snappykraken).

**`src/components/layout/theme-toggle.astro`:**
```astro
<button
  aria-label="Switch to dark mode"
  class="inline-flex size-10 cursor-pointer items-center justify-center rounded-full border border-line bg-page text-ink transition-colors hover:bg-page-2"
  data-theme-toggle
  type="button"
>
  <svg aria-hidden="true" class="js-sun-icon hidden size-5" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24">
    <circle cx="12" cy="12" r="4"></circle>
    <path d="M12 2v2M12 20v2M4.93 4.93l1.41 1.41M17.66 17.66l1.41 1.41M2 12h2M20 12h2M4.93 19.07l1.41-1.41M17.66 6.34l1.41-1.41"></path>
  </svg>
  <svg aria-hidden="true" class="js-moon-icon size-5" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24">
    <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>
  </svg>
</button>

<script is:inline>
;(function () {
  var KEY = 'PROJ-theme'
  var btn = document.querySelector('[data-theme-toggle]')
  var sun = btn && btn.querySelector('.js-sun-icon')
  var moon = btn && btn.querySelector('.js-moon-icon')
  if (!btn || !sun || !moon) return

  function update() {
    var dark = document.documentElement.classList.contains('dark')
    btn.setAttribute('aria-label', dark ? 'Switch to light mode' : 'Switch to dark mode')
    sun.classList.toggle('hidden', !dark)
    moon.classList.toggle('hidden', dark)
  }

  update()

  btn.addEventListener('click', function () {
    var next = !document.documentElement.classList.contains('dark')
    document.documentElement.classList.toggle('dark', next)
    try { localStorage.setItem(KEY, next ? 'dark' : 'light') } catch (e) {}
    update()
  })

  window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', function (e) {
    if (!localStorage.getItem(KEY)) {
      document.documentElement.classList.toggle('dark', e.matches)
      update()
    }
  })

  document.addEventListener('astro:after-swap', function () {
    var t = localStorage.getItem(KEY)
    if (t === 'dark' || (!t && matchMedia('(prefers-color-scheme:dark)').matches)) {
      document.documentElement.classList.add('dark')
    } else {
      document.documentElement.classList.remove('dark')
    }
    update()
  })
})()
</script>
```

**`src/pages/robots.txt.ts`:**
```typescript
import type { APIRoute } from 'astro'

export const GET: APIRoute = () => {
  const body = [
    'User-agent: *',
    'Allow: /',
    '',
    'Sitemap: https://TARGET.com/sitemap-index.xml',
    '',
  ].join('\n')

  return new Response(body, {
    headers: { 'Content-Type': 'text/plain; charset=utf-8' },
  })
}
```

**`public/_headers`:**
```
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=()
  Strict-Transport-Security: max-age=31536000; includeSubDomains
```

**`public/_redirects`:**
```
# Old URL -> New URL (301 permanent)
# /old-path /new-path 301
```

Compare original URLs vs clone URLs. Add 301s for any that changed. Handle trailing slash consistency -- set `trailingSlash` in astro.config to match original site's behavior.

---

## Phase 4: Build Pages

### Content strategy

**`src/data/content.ts`** for structured data:
```typescript
export const content = {
  meta: {
    url: 'https://target.com',
    siteName: 'Target Brand',
    title: 'Target Brand -- Tagline',
    description: 'Meta description under 160 chars',
    author: 'Target Brand',
    keywords: 'keyword1, keyword2',
  },
  nav: { items: [/* ... */] },
  footer: {
    columns: [/* ... */],
    legal: `© ${new Date().getFullYear()} Target Brand. All rights reserved.`,
    social: [/* ... */],
  },
  home: { hero: {/* ... */}, stats: [/* ... */], features: [/* ... */] },
  // Per-page sections...
} as const

export type Content = typeof content
```

**Markdown via Content Collections** for blog/case studies/long-form:

`src/content.config.ts` (in `src/` root, NOT inside `src/content/`):
```typescript
import { defineCollection, z } from 'astro:content'
import { glob } from 'astro/loaders'

const blog = defineCollection({
  loader: glob({ pattern: '**/*.md', base: './src/content/blog' }),
  schema: ({ image }) =>
    z.object({
      title: z.string(),
      description: z.string(),
      pubDate: z.coerce.date(),
      heroImage: image().optional(),
      heroImageAlt: z.string().default(''),
      author: z.string(),
      draft: z.boolean().default(false),
    }),
})

export const collections = { blog }
```

`src/pages/blog/[...slug].astro`:
```astro
---
import { getCollection, render } from 'astro:content'
import Layout from '@/layouts/base.astro'

export async function getStaticPaths() {
  const posts = await getCollection('blog', ({ data }) => !data.draft)
  return posts.map((post) => ({
    params: { slug: post.id },
    props: { post },
  }))
}

const { post } = Astro.props
const { Content } = await render(post)
---

<Layout title={post.data.title} description={post.data.description}>
  <article class="prose mx-auto">
    <h1>{post.data.title}</h1>
    <Content />
  </article>
</Layout>
```

**Decision rule:** Structured data (lists, grids, cards) -> `content.ts`. Prose (articles) -> markdown.

### Component patterns

**All `.astro` components** (zero JS). Only exceptions:
- Theme toggle (`<script is:inline>`)
- Mobile menu toggle (`<script is:inline>`)
- FAQ accordion (native `<details>`/`<summary>`)

**Mobile menu toggle:**
```astro
<button
  aria-expanded="false"
  aria-label="Open menu"
  class="md:hidden flex size-10 items-center justify-center"
  id="mobile-menu-toggle"
  type="button"
>
  <svg class="size-6" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
    <path d="M4 6h16M4 12h16M4 18h16" stroke-linecap="round" />
  </svg>
</button>

<div class="hidden md:hidden" id="mobile-menu">
  <!-- Nav links here -->
</div>

<script is:inline>
;(function () {
  var btn = document.getElementById('mobile-menu-toggle')
  var menu = document.getElementById('mobile-menu')
  if (!btn || !menu) return
  btn.addEventListener('click', function () {
    var open = !menu.classList.contains('hidden')
    menu.classList.toggle('hidden')
    btn.setAttribute('aria-expanded', String(!open))
    btn.setAttribute('aria-label', open ? 'Open menu' : 'Close menu')
  })
  document.addEventListener('astro:after-swap', function () {
    menu.classList.add('hidden')
    btn.setAttribute('aria-expanded', 'false')
  })
})()
</script>
```

**Naming:** kebab-case always. `hero-section.astro`, `nav.astro`, `page-hero.astro`.

**Active nav link:**
```astro
{nav.items.map((item) => (
  <a
    href={item.href}
    aria-current={Astro.url.pathname === item.href ? 'page' : undefined}
    class:list={[
      'transition-colors',
      Astro.url.pathname === item.href ? 'text-accent' : 'text-ink hover:text-accent',
    ]}
  >
    {item.label}
  </a>
))}
```

**CSS-only dropdown nav (no JS):**
```astro
<div class="group relative">
  <button class="flex items-center gap-1">{item.label}</button>
  <div class="invisible absolute left-0 top-full pt-2 opacity-0 transition-all group-hover:visible group-hover:opacity-100">
    <div class="rounded-lg border border-line bg-page p-2 shadow-lg">
      {item.children.map(child => <a href={child.href}>{child.label}</a>)}
    </div>
  </div>
</div>
```

**Logo dark/light variant:**
```astro
<img src="/images/logos/logo.svg" alt="Logo" class="h-8 dark:hidden" />
<img src="/images/logos/logo-white.svg" alt="Logo" class="hidden h-8 dark:block" />
```

### Page build order

1. Base layout + nav + footer + theme toggle
2. Design system page
3. Homepage (most sections)
4. Secondary pages
5. Blog/case study templates (if applicable)
6. 404 page
7. Robots.txt, sitemap (auto-generated)

### SEO per page

Every page MUST have:
- `<title>` (unique)
- `<meta name="description">` (unique, < 160 chars)
- `<meta name="author">`
- `<meta name="keywords">`
- `<meta name="color-scheme" content="light dark">`
- Canonical URL
- Open Graph: `og:title`, `og:description`, `og:url`, `og:type`, `og:image`, `og:image:width`, `og:image:height`, `og:site_name`, `og:locale`
- Twitter: `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`
- Theme-color with light/dark media queries
- `<link rel="sitemap">` pointing to `/sitemap-index.xml`

### JSON-LD structured data

Inject via named slot pattern:
```astro
<!-- In page file -->
<Layout title="Home">
  <Fragment slot="head">
    <script type="application/ld+json" set:html={JSON.stringify(orgSchema)} />
    <script type="application/ld+json" set:html={JSON.stringify(websiteSchema)} />
    <script type="application/ld+json" set:html={JSON.stringify(faqSchema)} />
  </Fragment>
  <!-- page content -->
</Layout>
```

**Schema catalog -- use when applicable:**

| Schema | When |
|---|---|
| `Organization` | Homepage (always) |
| `WebSite` | Homepage (always) |
| `FAQPage` | Any page with FAQ section |
| `BreadcrumbList` | Pages with depth > 1 |
| `Article`/`BlogPosting` | Blog posts |
| `LocalBusiness` | Contact/location pages |
| `SoftwareApplication` | SaaS pricing pages |
| `Person` | Team/bio pages |

### Third-party embeds

| Embed | Approach |
|---|---|
| YouTube/Vimeo | `lite-youtube`/`lite-vimeo` web component (lazy, tiny JS) |
| Google Maps | Static map image linking to Google Maps, or `<iframe loading="lazy">` |
| Calendly/booking | `<iframe loading="lazy">` |
| Social feeds | Static snapshot, link to profile |
| Chat widgets | Load on interaction (click-to-load), not page load |
| Analytics | `@astrojs/partytown` to offload to web worker |

### Forms

Options for form backends (static site has no server):
1. **Formspree** / **Formspark** -- simple, hosted form endpoint
2. **Cloudflare Worker** -- custom endpoint if complex logic needed
3. **mailto: link** -- simplest, no backend needed
4. At minimum: form UI renders correctly, submit shows a thank-you state. Never broken/erroring forms.

### 404 page

```astro
---
import Layout from '@/layouts/base.astro'
---

<Layout title="Page Not Found" description="The page you're looking for doesn't exist.">
  <section class="flex min-h-[60vh] flex-col items-center justify-center text-center">
    <h1 class="text-6xl font-bold text-ink">404</h1>
    <p class="mt-4 text-lg text-muted">Page not found</p>
    <a href="/" class="mt-8 rounded-lg bg-accent px-6 py-3 text-white transition-colors hover:bg-accent-dark">
      Back to home
    </a>
  </section>
</Layout>
```

Cloudflare Pages auto-serves this for missing routes.

### Animations and carousels

**No animation libraries.** Use native APIs and CSS only:

| Pattern | Approach |
|---|---|
| Carousel/slider | CSS `scroll-snap-type: x mandatory` + `scroll-snap-align: start`. Native scroll. Zero JS. |
| Auto-advancing carousel | CSS `@keyframes` with `animation` to scroll. Or minimal `<script is:inline>` with `setInterval` + `scrollTo`. |
| Fade in on scroll | CSS `@starting-style` + `transition` (modern). Or `IntersectionObserver` in `<script is:inline>` adding a class. |
| Hover effects | Tailwind `transition-*` + `hover:` utilities. |
| Accordion | Native `<details>`/`<summary>`. Zero JS. |
| Tabs | CSS `:target` selector or radio-button hack. Or minimal inline JS toggling `hidden`. |
| Smooth scroll | CSS `scroll-behavior: smooth` (already in `app.css`). |
| Progress bars | CSS `@keyframes` or `width` transition. |
| Parallax | `background-attachment: fixed`. No JS parallax libraries. |
| Video background | `<video autoplay muted loop playsinline poster="frame.jpg">`. Static poster as fallback. |
| Lottie animations | Replace with static frame or CSS equivalent. If critical: inline the lottie-player as an exception to zero-JS rule, document in CLONE-WIKI.md. |

Always respect `prefers-reduced-motion`: disable or reduce all animations.

### Internationalization (i18n)

Default: single language (English). Structure supports multi-language if needed.

**Single language (default):**
```
src/pages/
├── index.astro
├── about.astro
└── blog/[...slug].astro
src/data/content.ts         # English content
```

**Multi-language (when customer needs it):**
```
src/pages/
├── index.astro             # Default language
├── [lang]/                 # Other languages
│   ├── index.astro
│   ├── about.astro
│   └── blog/[...slug].astro
src/content/
├── blog/en/                # English markdown
│   └── post-1.md
├── blog/es/                # Spanish markdown
│   └── post-1.md
src/data/
├── content.ts              # Default (English)
├── content.es.ts           # Spanish overrides
└── i18n.ts                 # Language config + t() helper
```

Add `hreflang` tags when multi-language:
```html
<link rel="alternate" hreflang="en" href="https://target.com/" />
<link rel="alternate" hreflang="es" href="https://target.com/es/" />
<link rel="alternate" hreflang="x-default" href="https://target.com/" />
```

Set `<html lang="xx">` per page. Add language switcher in nav.

---

## Phase 5: Optimize, Audit, and Ship

**REQUIRED:** Invoke the `site-optimizer` skill for all optimization, auditing, and shipping.

The `site-optimizer` skill covers:
- Image optimization (Astro `<Image>`, responsive `widths`/`sizes`, `fetchpriority`)
- Font subsetting and self-hosting (pyftsubset, WOFF2, `@font-face`)
- SEO checklist (meta tags, JSON-LD, Open Graph, sitemaps)
- Accessibility checklist (skip-to-content, focus-visible, WCAG AAA)
- Performance CSS (scrollbar, selection, tap-highlight, font smoothing)
- Cloudflare Pages headers (`_headers`, `_redirects`)
- Audit loop (unlighthouse-ci + seomator + pa11y -- all headless)
- Visual comparison (odiff + magick for pixel-perfect feedback)
- Pre-ship checklist and deploy

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

Create/update after every session:
- Decisions and rationale
- Issues resolved
- Current audit scores per page
- Remaining work
- Font licensing notes
- Dark mode derivation notes
- Deviations from original and why
- Customer feedback log

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
