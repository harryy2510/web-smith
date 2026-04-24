# Audit Tools and Workflow

## `unlighthouse.config.ts`

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

## Tool Reference

All tools run **headless** (no browser UI, no interactive prompts). Safe for autonomous agents and CI.

| Tool | Covers | Headless | Install |
|---|---|---|---|
| `unlighthouse-ci` | Lighthouse per-page (performance, a11y, SEO, best practices) | Yes (default) | devDep |
| `seomator` | 251 rules: deep SEO, AEO/GEO readiness, structured data, AI bot access, llms.txt | Yes (CLI-only) | `bun add -g @seomator/seo-audit` |
| `pa11y` | Deep WCAG accessibility (axe-core engine, WCAG2AA/AAA) | Yes (headless Chromium) | `bun add -g pa11y` |
| `shot-scraper` | Screenshots, HTML fetch, HAR extract, JS execution | Yes (headless Playwright) | `pip install shot-scraper` |
| `odiff` | Pixel-level image diff with percentage score | Yes (CLI-only) | `bun add -g odiff-bin` |
| `magick compare` | Color extraction, SSIM, structural diff | Yes (CLI-only) | `brew install imagemagick` |

## Workflow

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

## Fix Loop

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
