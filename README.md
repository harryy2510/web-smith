<p align="center">
  <h1 align="center">Web Smith</h1>
  <p align="center">
    <strong>Give it a URL. Get back Lighthouse 100.</strong>
  </p>
  <p align="center">
    <code>2 skills</code> &middot; <code>7 audit tools</code> &middot; <code>all headless</code> &middot; <code>pixel-perfect</code>
  </p>
</p>

<p align="center">
  <a href="https://astro.build"><img src="https://img.shields.io/badge/Astro_6-BC52EE?style=flat-square&logo=astro&logoColor=white" alt="Astro"></a>
  <a href="https://tailwindcss.com"><img src="https://img.shields.io/badge/Tailwind_v4-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white" alt="Tailwind v4"></a>
  <a href="https://pages.cloudflare.com"><img src="https://img.shields.io/badge/Cloudflare_Pages-F38020?style=flat-square&logo=cloudflare&logoColor=white" alt="Cloudflare Pages"></a>
  <a href="https://claude.ai"><img src="https://img.shields.io/badge/Claude_Code-CC785C?style=flat-square&logo=anthropic&logoColor=white" alt="Claude Code"></a>
  <a href="https://excalidraw.com"><img src="https://img.shields.io/badge/Excalidraw-6965DB?style=flat-square&logo=excalidraw&logoColor=white" alt="Excalidraw"></a>
</p>

---

> Distributed through the [claude-toolkit](https://github.com/harryy2510/claude-toolkit) marketplace.

```
claude plugin marketplace add harryy2510/claude-toolkit
claude plugin install web-smith@claude-toolkit
```

---

<p align="center">
  <img src="docs/web-smith-flow.png" alt="Web Smith Flow" />
</p>

---

## What This Does

You point an AI agent at a marketing website. It scrapes everything, extracts the design system, rebuilds it as a static site, then beats the original on every metric.

The customer gets their own site back -- same look, 10x faster, 100/100/100/100 Lighthouse, dark mode, responsive, deployed on Cloudflare Pages.

---

## Two Skills

| | Skill | Scope | Trigger |
|---|---|---|---|
| 🔨 | **website-cloner** | Scrape + rebuild a customer site as Astro static | "clone this site", "rebuild this website" |
| ⚡ | **site-optimizer** | Optimize any static site to Lighthouse 100 | "optimize this site", "fix Lighthouse score" |

`website-cloner` does the full pipeline then invokes `site-optimizer` at the end. `site-optimizer` works standalone on any static site -- not just clones, not just Astro.

---

## website-cloner

Clones a customer's marketing website into a high-performance Astro static site.

**Phase 1 -- Scrape.** `shot-scraper` fetches HTML, takes screenshots at 3 viewports (desktop/tablet/mobile), downloads all assets via HAR extraction. `dembrandt` extracts design tokens (colors, fonts, spacing). Overlay removal JS strips cookie banners, modals, chat widgets before capture.

**Phase 2 -- Design System.** Maps scraped colors to semantic CSS variables (`--page`, `--ink`, `--accent`). Derives dark mode via OKLCH lightness inversion even if original has none. Builds a `/design-system` page showing all tokens, typography, component states.

**Phase 3 -- Scaffold.** Creates Astro 6 project with Tailwind v4, ESLint, Prettier, Husky, View Transitions, theme toggle, skip-to-content, mobile menu. Full config files provided -- `astro.config.ts`, `eslint.config.ts`, `prettier.config.ts`, `.gitignore`, `_headers`, `_redirects`.

**Phase 4 -- Build Pages.** Content in `content.ts` (structured) or markdown via Content Collections (prose). All `.astro` components, zero JS by default. SEO: JSON-LD schemas, Open Graph, sitemaps. Responsive always. i18n-ready structure. Native CSS animations, `scroll-snap` carousels, `<details>` accordions.

**Phase 5 -- Optimize + Audit + Ship.** Invokes `site-optimizer`.

### Stack

```
Astro 6 (static)  +  Tailwind CSS v4  +  Cloudflare Pages
```

### Hard Rules

- Zero JS by default (only `<script is:inline>` for theme + mobile menu)
- Dark mode always (even if original has none)
- Responsive always (even if original is not)
- Self-hosted fonts (never Google Fonts CDN)
- All images via Astro `<Image>` (AVIF/WebP, width/height/alt)
- Content separated (`content.ts` or markdown, never hardcoded)
- Lighthouse 100/100/100/100 target

---

## site-optimizer

Framework-agnostic. Works on Astro, Next.js, Hugo, or any static site.

### What It Covers

| Area | What |
|---|---|
| **Images** | Astro `<Image>`, responsive `widths`/`sizes`, `fetchpriority="high"` on LCP, lazy everything else |
| **Fonts** | `pyftsubset` to Latin WOFF2, `font-display: swap`, preload critical font, variable fonts for 3+ weights |
| **SEO** | Meta tags, canonical, Open Graph, Twitter cards, JSON-LD catalog (Organization, FAQ, Article, LocalBusiness, etc.) |
| **Accessibility** | Skip-to-content, `focus-visible`, `aria-current`, color contrast AAA (7:1), touch targets 44px, `prefers-reduced-motion` |
| **Performance CSS** | Font smoothing, `text-rendering`, scrollbar styling, `::selection`, `-webkit-tap-highlight-color`, view transition reduced-motion |
| **Security** | `_headers` / `_redirects` (platform-agnostic reference for Cloudflare, Netlify, Vercel, Nginx, Apache) |

### Audit Tools (All Headless)

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  unlighthouse-ci     Lighthouse per-page (performance, a11y, SEO)    │
│  seomator            251 SEO rules + AEO/GEO readiness               │
│  pa11y               WCAG accessibility (axe-core, WCAG2AA/AAA)      │
│  shot-scraper        Screenshots, HTML fetch, JS execution           │
│  odiff               Pixel-level image diff + percentage score       │
│  magick compare      Color extraction at specific coordinates        │
│  excalidraw-cli      Diagram generation + PNG/SVG export             │
│                                                                      │
│  All run headless. No browser UI. Safe for autonomous agents + CI.   │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### Visual Comparison (Pixel-Perfect Feedback)

The real gap in AI-built sites: "close enough" instead of "pixel perfect." Web Smith fixes this with element-level screenshot diffs:

```
  shot-scraper original.com -s "nav" -o original-nav.png
  shot-scraper localhost:4321 -s "nav" -o clone-nav.png
  odiff original-nav.png clone-nav.png diff-nav.png --parsable-stdout

  → "2400;2.00"  (2% diff -- needs fixing)

  magick original-nav.png -crop 1x1+200+50 txt:
  → #1B1D21

  magick clone-nav.png -crop 1x1+200+50 txt:
  → #2A2824   ← wrong color, fix the --ink token
```

Per-section diffs (nav, hero, footer, full page). Exact color extraction at pixel coordinates. Specific feedback: "heading is 4px too tall", "button is #FF0000 but should be #EE0000".

---

## Prerequisites

```bash
pip install shot-scraper && shot-scraper install
bun add -g dembrandt odiff-bin @seomator/seo-audit pa11y @swiftlysingh/excalidraw-cli
brew install imagemagick
```

---

## How It Works

### Clone a website

```
You:  "Clone snappykraken.com"

  1. Scrapes all pages ── shot-scraper: HTML, screenshots, assets
  2. Extracts design   ── dembrandt: colors, fonts, spacing
  3. Scaffolds project  ── Astro 6 + Tailwind v4 + ESLint + Husky
  4. Builds every page  ── content.ts, JSON-LD, dark mode, responsive
  5. Optimizes + audits ── site-optimizer: images, fonts, SEO, a11y
  6. Visual diffs       ── odiff: per-section pixel comparison
  7. Ships              ── Cloudflare Pages, preview URL, custom domain
```

### Optimize any existing site

```
You:  "Optimize this site for Lighthouse 100"

  1. Audits    ── unlighthouse-ci + seomator + pa11y
  2. Identifies ── failing pages, categories, specific issues
  3. Fixes     ── images, fonts, meta, contrast, CLS, LCP
  4. Re-audits  ── repeat until 100/100/100/100
  5. Ships     ── deploy to any platform
```

---

## Project Structure

```
web-smith/
├── plugins/web-smith/
│   ├── .claude-plugin/plugin.json
│   └── skills/
│       ├── website-cloner/SKILL.md    ← scrape + rebuild pipeline
│       └── site-optimizer/SKILL.md    ← optimization + audit loop
├── docs/
│   ├── web-smith-flow.excalidraw      ← editable diagram
│   └── web-smith-flow.png             ← rendered diagram
└── README.md
```

---

## Install

```bash
# Add marketplace (one time)
claude plugin marketplace add harryy2510/claude-toolkit

# Install
claude plugin install web-smith@claude-toolkit
```

---

## Author

**Hariom Sharma** -- [github.com/harryy2510](https://github.com/harryy2510)

## License

MIT
