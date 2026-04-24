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

**Read `references/audit-tools.md`** for: unlighthouse config, tool reference table, workflow commands, fix loop, and common fixes table.

---

## 8. Visual Comparison (for clone projects)

**Read `references/visual-comparison.md`** for: element-level screenshots with shot-scraper, odiff pixel diffing, magick color extraction, and the fix-and-re-diff workflow.

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
