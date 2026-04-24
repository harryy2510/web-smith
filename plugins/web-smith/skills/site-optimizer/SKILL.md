---
name: site-optimizer
description: "Use when optimizing any static site for Lighthouse 100, SEO, AEO, accessibility, and performance. Triggers: optimize site, improve Lighthouse score, fix SEO, fix accessibility, audit site performance, pagespeed optimization, WCAG compliance, AEO readiness, Core Web Vitals. Framework-agnostic -- works on Astro, Next.js, Hugo, or any static site. Image examples use Astro syntax but concepts apply universally."
user_invocable: true
---

# Site Optimizer

Optimize any static site to Lighthouse 100/100/100/100. Covers images, fonts, SEO, accessibility, AEO/GEO readiness, and performance. Framework-agnostic -- image examples use Astro syntax but all audit tools, font optimization, SEO checklists, and visual diffing work on any static site.

## When to Use

- Existing Astro site needs Lighthouse score improvement
- SEO/AEO audit and fix cycle
- WCAG accessibility compliance
- Performance optimization (LCP, CLS, TBT, INP)
- After building a cloned site (auto-invoked by `website-cloner` skill)
- Any Astro project needing production polish

## Prerequisites

```bash
pip install shot-scraper && shot-scraper install
bun add -g odiff-bin @seomator/seo-audit pa11y
bun add -D unlighthouse
# imagemagick via: brew install imagemagick
```

---

## 1. Image Optimization

ALL raster images in `src/assets/images/` (never `public/` for rasters). Astro optimizes at build time (AVIF/WebP).

**Single image:**
```astro
---
import { Image } from 'astro:assets'
import heroImage from '@/assets/images/heroes/home-hero.jpg'
---

<Image
  alt="Descriptive alt text"
  src={heroImage}
  width={1200}
  height={630}
  loading="eager"
  decoding="async"
  fetchpriority="high"
  class="w-full object-cover"
/>
```

Use `fetchpriority="high"` ONLY on LCP image.

**Responsive hero image (large screens):**
```astro
<Image
  src={heroImage}
  widths={[640, 1024, 1920]}
  sizes="(max-width: 640px) 640px, (max-width: 1024px) 1024px, 1920px"
  alt="Hero"
  loading="eager"
  fetchpriority="high"
/>
```

**Dynamic image collection:**
```astro
---
import { Image } from 'astro:assets'
import { type ImageMetadata } from 'astro'

const images = import.meta.glob<{ default: ImageMetadata }>(
  '/src/assets/images/projects/*.{webp,png,jpg,avif}',
  { eager: true },
)
---

{items.map((item) => {
  const filename = item.image.split('/').pop()!
  const key = `/src/assets/images/projects/${filename}`
  const img = images[key]
  return img ? (
    <Image alt={item.name} src={img.default} loading="lazy" class="w-full object-contain" />
  ) : (
    <img alt={item.name} src={item.image} loading="lazy" class="w-full object-contain" />
  )
})}
```

**Rules:**
- `loading="eager"` + `fetchpriority="high"` ONLY for LCP image
- `width` + `height` required (CLS prevention)
- `alt` required (accessibility)
- SVGs in `public/images/logos/` (no optimization needed)
- For 50+ images on one page: use smaller widths for thumbnails, consider pagination

---

## 2. Font Optimization

**If using custom fonts:**

1. Download (Google Fonts, fontsource, or existing assets)
2. Subset to Latin:
```bash
pip3 install fonttools brotli
pyftsubset Font-Regular.ttf \
  --output-file=public/fonts/Font-Regular-latin.woff2 \
  --flavor=woff2 \
  --unicodes="U+0000-00FF,U+0131,U+0152-0153,U+02BB-02BC,U+02C6,U+02DA,U+02DC,U+0304,U+0308,U+0329,U+2000-206F,U+2074,U+20AC,U+2122,U+2191,U+2193,U+2212,U+2215,U+FEFF,U+FFFD" \
  --layout-features="kern,liga,calt,ccmp,locl" \
  --no-hinting --desubroutinize
```
3. If 3+ weights: prefer variable font (single file, all weights)
4. WOFF2 only (97%+ support, no WOFF fallback needed)
5. Create `src/styles/fonts.css`:
```css
@font-face {
  font-family: 'FontName';
  font-style: normal;
  font-weight: 100 900; /* variable font range, or single weight e.g. 400 */
  font-display: swap;
  src: url('/fonts/FontName-latin.woff2') format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6,
    U+02DA, U+02DC, U+2000-206F, U+20AC, U+2122, U+2191, U+2193, U+2212,
    U+2215, U+FEFF, U+FFFD;
}
```
6. Import in `app.css`: `@import './fonts.css';` (remove if using system fonts)
7. Preload critical font in `<head>`:
```html
<link rel="preload" href="/fonts/FontName-latin.woff2" as="font" type="font/woff2" crossorigin />
```
8. Never load from Google Fonts CDN in production. Always self-host.

---

## 3. SEO Checklist

Every page MUST have:
- `<title>` (unique)
- `<meta name="description">` (unique, < 160 chars)
- `<meta name="author">`
- `<meta name="keywords">`
- `<meta name="color-scheme" content="light dark">`
- Canonical URL
- Open Graph: `og:title`, `og:description`, `og:url`, `og:type`, `og:image`, `og:image:width`, `og:image:height`, `og:site_name`, `og:locale`
- Twitter: `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`
- Theme-color with light/dark media queries:
```html
<meta content="#ffffff" media="(prefers-color-scheme: light)" name="theme-color" />
<meta content="#121214" media="(prefers-color-scheme: dark)" name="theme-color" />
```
- `<link rel="sitemap">` pointing to `/sitemap-index.xml`
- `<link rel="icon">` (SVG + ICO) + `<link rel="apple-touch-icon">`

### JSON-LD structured data

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

Inject via named slot:
```astro
<Layout title="Home">
  <Fragment slot="head">
    <script type="application/ld+json" set:html={JSON.stringify(schema)} />
  </Fragment>
</Layout>
```

---

## 4. Accessibility Checklist

- Skip-to-content link:
```html
<a class="sr-only focus:not-sr-only focus:fixed focus:left-4 focus:top-4 focus:z-50 focus:rounded-full focus:bg-ink focus:px-4 focus:py-2 focus:text-page focus:outline-none" href="#main-content">
  Skip to content
</a>
```
- Semantic HTML: `<nav>`, `<main>`, `<article>`, `<section>`, `<footer>`
- All images have `alt` text
- ARIA labels on interactive elements
- `aria-current="page"` on active nav links
- Focus-visible indicators (in `app.css`):
```css
:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}
```
- `prefers-reduced-motion` respected:
```css
@media (prefers-reduced-motion: reduce) {
  html { scroll-behavior: auto; }
  ::view-transition-group(*),
  ::view-transition-old(*),
  ::view-transition-new(*) {
    animation-duration: 0s !important;
  }
}
```
- Color contrast AAA (7:1 minimum)
- Touch targets 44px minimum

---

## 5. Performance CSS

Essential `app.css` patterns for Lighthouse 100:

```css
html {
  scroll-behavior: smooth;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  scrollbar-color: color-mix(in oklab, var(--line) 80%, transparent) transparent;
  scrollbar-width: thin;
}

body {
  -webkit-tap-highlight-color: transparent;
  text-rendering: optimizeLegibility;
}

::selection {
  background: var(--accent);
  color: #fff;
}

::-webkit-scrollbar { width: 6px; }
::-webkit-scrollbar-track { background: transparent; }
::-webkit-scrollbar-thumb {
  background: color-mix(in oklab, var(--line) 80%, transparent);
  border-radius: 3px;
}
```

---

## 6. Security Headers and Redirects

These headers should be set regardless of hosting platform. Implementation varies:

| Platform | Headers file | Redirects file |
|---|---|---|
| Cloudflare Pages | `public/_headers` | `public/_redirects` |
| Netlify | `public/_headers` | `public/_redirects` |
| Vercel | `vercel.json` (`headers` array) | `vercel.json` (`redirects` array) |
| Nginx | `nginx.conf` (`add_header`) | `nginx.conf` (`rewrite`) |
| Apache | `.htaccess` (`Header set`) | `.htaccess` (`Redirect 301`) |
| Any CDN | CDN dashboard / edge rules | CDN dashboard / edge rules |

**Required security headers:**
```
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

**Redirects:** Map all old URLs to new URLs with 301 permanent redirects. Check trailing slash consistency.

---

## 7. Audit Loop

### `unlighthouse.config.ts`:
```typescript
export default {
  site: 'http://localhost:UNIQUE_PORT',
  ci: {
    budget: {
      performance: 100,
      accessibility: 100,
      'best-practices': 100,
      seo: 100,
    },
    reporter: 'jsonExpanded',
  },
  scanner: { device: 'desktop', throttle: true },
  outputPath: '.unlighthouse',
}
```

### Audit tools

All tools run **headless** (no browser UI, no interactive prompts). Safe for autonomous agents and CI.

| Tool | Covers | Headless | Install |
|---|---|---|---|
| `unlighthouse-ci` | Lighthouse per-page (performance, a11y, SEO, best practices) | Yes (default) | devDep |
| `seomator` | 251 rules: deep SEO, AEO/GEO readiness, structured data, AI bot access, llms.txt | Yes (CLI-only) | `bun add -g @seomator/seo-audit` |
| `pa11y` | Deep WCAG accessibility (axe-core engine, WCAG2AA/AAA) | Yes (headless Chromium) | `bun add -g pa11y` |
| `shot-scraper` | Screenshots, HTML fetch, HAR extract, JS execution | Yes (headless Playwright) | `pip install shot-scraper` |
| `odiff` | Pixel-level image diff with percentage score | Yes (CLI-only) | `bun add -g odiff-bin` |
| `magick compare` | Color extraction, SSIM, structural diff | Yes (CLI-only) | `brew install imagemagick` |

### Workflow

```bash
# Build + preview
bun run build && bun run preview &
sleep 2

# 1. Lighthouse audit (all pages, scores + budgets)
bunx unlighthouse-ci

# 2. Deep SEO + AEO audit (crawls all linked pages)
seomator audit http://localhost:UNIQUE_PORT --crawl -m 50 --format json -o .audit/seo-report.json

# 3. WCAG accessibility audit (per page)
pa11y http://localhost:UNIQUE_PORT --runner axe --standard WCAG2AA --chromium-arg=--no-sandbox -r json > .audit/a11y-home.json
# Repeat for each page

kill %1

# Review results
cat .unlighthouse/ci-result.json
cat .audit/seo-report.json
cat .audit/a11y-home.json
```

Add `.audit/` to `.gitignore`.

### Fix loop

Target: Lighthouse 100/100/100/100 every page + zero SEO errors + zero WCAG violations. If not achievable after 200 attempts, accept 99+ and report to human.

**Common fixes:**

| Issue | Fix |
|---|---|
| LCP high | Preload hero image with `fetchpriority="high"`, responsive `widths`/`sizes` |
| CLS > 0 | width/height on all images, reserve space for fixed nav, no dynamic above-fold content |
| Missing alt | Add descriptive alt to all `<img>` and `<Image>` |
| Low contrast | Adjust tokens (7:1 AAA minimum) |
| Missing meta | og:image, description, canonical, sitemap link |
| Render-blocking | Move scripts to bottom, use `is:inline` or partytown |
| TBT > 0 | Remove/defer JS, partytown for analytics |
| INP issues | Ensure toggle handlers are lightweight and synchronous |

---

## 8. Visual Comparison (for clone projects)

When comparing against an original site, use element-level screenshot diffs for pixel-perfect feedback.

**Step 1: Element-level screenshots**

```bash
# Original
shot-scraper "https://ORIGINAL.com" -s "nav" -o compare/original-nav.png
shot-scraper "https://ORIGINAL.com" -s "main > section:first-child" -o compare/original-hero.png
shot-scraper "https://ORIGINAL.com" -s "footer" -o compare/original-footer.png
shot-scraper "https://ORIGINAL.com" -o compare/original-full.png --full-page --width 1280

# Clone
shot-scraper "http://localhost:UNIQUE_PORT" -s "nav" -o compare/clone-nav.png
shot-scraper "http://localhost:UNIQUE_PORT" -s "main > section:first-child" -o compare/clone-hero.png
shot-scraper "http://localhost:UNIQUE_PORT" -s "footer" -o compare/clone-footer.png
shot-scraper "http://localhost:UNIQUE_PORT" -o compare/clone-full.png --full-page --width 1280
```

**Step 2: Quick diff score per section**

```bash
odiff compare/original-nav.png compare/clone-nav.png compare/diff-nav.png --parsable-stdout
odiff compare/original-hero.png compare/clone-hero.png compare/diff-hero.png --parsable-stdout
odiff compare/original-footer.png compare/clone-footer.png compare/diff-footer.png --parsable-stdout
odiff compare/original-full.png compare/clone-full.png compare/diff-full.png --parsable-stdout
# Output: diffPixels;diffPercentage  (e.g., "2400;2.00")
```

If any section > 2% diff, fix it. Read the diff PNG to see exactly what's wrong.

**Step 3: Extract specific color differences**

```bash
magick compare/original-hero.png -crop 1x1+200+165 txt:
# Output: 0,0: (238,0,0,255) #EE0000FF

magick compare/clone-hero.png -crop 1x1+200+165 txt:
# Output: 0,0: (255,0,0,255) #FF0000FF
```

**Step 4: Read diff images visually**

Read diff PNGs with the Read tool. Produce specific feedback:
- "Nav height is 4px taller than original"
- "Hero heading font-weight is 600 but should be 700"
- "Button background is #FF0000 but should be #EE0000"

**Step 5: Fix and re-diff** until all sections < 1% diff.

**Fonts will cause differences.** If using different fonts, focus on layout, spacing, colors, and structure instead of text rendering.

---

## 9. Pre-ship Checklist

- [ ] `bun run build` zero warnings
- [ ] `bunx unlighthouse-ci` passes (100/100/100/100)
- [ ] `seomator audit` zero errors
- [ ] `pa11y` zero WCAG violations
- [ ] Sitemap + robots.txt correct
- [ ] JSON-LD validates
- [ ] Custom 404 page works
- [ ] `_redirects` file complete
- [ ] `_headers` file has security headers
- [ ] No console errors
- [ ] No dead links
- [ ] Responsive at 375px, 768px, 1440px
- [ ] Dark mode working (if applicable)
- [ ] `prefers-reduced-motion` respected
- [ ] Favicons: SVG + ICO + apple-touch-icon + og-image

### Deploy

1. Push to GitHub
2. Connect to hosting platform (Cloudflare Pages, Netlify, Vercel, etc.)
3. Set build command and output directory (typically `dist/` or `build/`)
4. Add custom domain + verify SSL
5. Share preview URL with stakeholders before going live

---

## Quick Reference

| Task | Command |
|---|---|
| Build | `bun run build` |
| Preview | `bun run preview` |
| Check | `bun run check` |
| Fix | `bun run fix` |
| Lighthouse audit | `bunx unlighthouse-ci` |
| SEO audit | `seomator audit URL --crawl -m 50 --format json -o .audit/seo.json` |
| WCAG audit | `pa11y URL --runner axe --standard WCAG2AA -r json` |
| Screenshot | `shot-scraper URL -o file.png --full-page` |
| Element screenshot | `shot-scraper URL -s "nav" -o nav.png` |
| Pixel diff | `odiff original.png clone.png diff.png --parsable-stdout` |
| Color at pixel | `magick img.png -crop 1x1+X+Y txt:` |
| Subset font | `pyftsubset font.ttf --output-file=out.woff2 --flavor=woff2 --unicodes="U+0-FF" --no-hinting` |
