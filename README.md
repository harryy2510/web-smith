<p align="center">
  <h1 align="center">Web Smith</h1>
  <p align="center">
    <strong>Clone any marketing website into a Lighthouse-perfect static site.</strong>
  </p>
  <p align="center">
    <code>2 skills</code> &middot; <code>0 dependencies</code> &middot; <code>Lighthouse 100</code>
  </p>
</p>

<p align="center">
  <a href="https://astro.build"><img src="https://img.shields.io/badge/Astro_6-BC52EE?style=flat-square&logo=astro&logoColor=white" alt="Astro"></a>
  <a href="https://tailwindcss.com"><img src="https://img.shields.io/badge/Tailwind_v4-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white" alt="Tailwind v4"></a>
  <a href="https://pages.cloudflare.com"><img src="https://img.shields.io/badge/Cloudflare_Pages-F38020?style=flat-square&logo=cloudflare&logoColor=white" alt="Cloudflare Pages"></a>
  <a href="https://claude.ai"><img src="https://img.shields.io/badge/Claude_Code-CC785C?style=flat-square&logo=anthropic&logoColor=white" alt="Claude Code"></a>
</p>

---

> **Note:** Web Smith is distributed through the [claude-toolkit](https://github.com/harryy2510/claude-toolkit) marketplace.

```
+--------------------------------------------------------------------------+
|                                                                          |
|   claude plugin marketplace add harryy2510/claude-toolkit                |
|   claude plugin install web-smith@claude-toolkit                         |
|                                                                          |
|   Give it a URL. Get back a pixel-perfect, Lighthouse 100, dark mode,    |
|   responsive static site deployed on Cloudflare Pages.                   |
|                                                                          |
+--------------------------------------------------------------------------+
```

---

## Skills

Loaded on-demand. Only the relevant skill enters context.

| Skill | What it does | Trigger |
|-------|-------------|---------|
| **website-cloner** | Full pipeline: scrape site, extract design tokens, scaffold Astro project, build pixel-perfect pages, optimize, audit, ship | "clone this site", "rebuild this website", "migrate from WordPress" |
| **site-optimizer** | Framework-agnostic optimization: Lighthouse 100, SEO, AEO, WCAG accessibility, visual diff, font subsetting, security headers | "optimize this site", "fix Lighthouse score", "audit SEO" |

### website-cloner

Clones a customer's marketing website into a high-performance Astro static site.

```
1. Scrape  -->  2. Design System  -->  3. Scaffold  -->  4. Build Pages  -->  5. Optimize (site-optimizer)
     |                  |                     |                  |                       |
  shot-scraper      dembrandt            Astro 6+         content.ts            Lighthouse 100
  screenshots       color tokens         Tailwind v4      dark mode             SEO + AEO
  HTML + assets     typography           ESLint/Prettier   responsive            WCAG a11y
  site metadata     dark mode derive     Husky hooks       i18n ready            visual diff
```

**Stack:** Astro 6 + Tailwind CSS v4 + Cloudflare Pages

**Tools:** shot-scraper, dembrandt, odiff, unlighthouse-ci, seomator, pa11y, pyftsubset, imagemagick

### site-optimizer

Optimizes any static site to Lighthouse 100/100/100/100. Framework-agnostic -- works on Astro, Next.js, Hugo, or any static site.

```
1. Images  -->  2. Fonts  -->  3. SEO  -->  4. Accessibility  -->  5. Audit Loop  -->  6. Ship
     |              |             |               |                      |
  AVIF/WebP     subset        JSON-LD          skip-to-content      unlighthouse-ci
  responsive    self-host     Open Graph       focus-visible        seomator (AEO)
  lazy load     WOFF2 only    meta tags        WCAG AAA             pa11y (WCAG)
  fetchpriority swap          sitemaps         reduced-motion       visual diff
```

**Audit tools (all headless):** unlighthouse-ci, seomator (251 SEO rules + AEO), pa11y (WCAG), odiff (pixel diff), imagemagick (color extraction)

---

## Prerequisites

```bash
# Scraping + visual diff
pip install shot-scraper && shot-scraper install
bun add -g dembrandt odiff-bin @seomator/seo-audit pa11y
brew install imagemagick

# Per-project (added by website-cloner during scaffold)
bun add -D unlighthouse
```

---

## How It Works

### Clone a website

```
You: "Clone snappykraken.com"

Web Smith:
  1. Scrapes all pages (shot-scraper: HTML, screenshots at 3 viewports, assets via HAR)
  2. Extracts design tokens (dembrandt: colors, fonts, spacing)
  3. Scaffolds Astro project (Tailwind v4, ESLint, Prettier, Husky, dark mode)
  4. Builds every page pixel-perfect (content.ts, semantic HTML, JSON-LD, responsive)
  5. Invokes site-optimizer for audit loop
  6. Ships to Cloudflare Pages
```

### Optimize any existing site

```
You: "Optimize this site for Lighthouse 100"

Web Smith:
  1. Audits with unlighthouse-ci + seomator + pa11y
  2. Identifies failing pages and categories
  3. Fixes issues (images, fonts, meta, contrast, CLS, LCP)
  4. Re-audits until 100/100/100/100
  5. Visual diff against original (if clone project)
```

---

## Author

**Hariom Sharma** -- [github.com/harryy2510](https://github.com/harryy2510)

---

## License

MIT
