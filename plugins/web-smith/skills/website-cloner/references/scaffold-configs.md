# Scaffold Config Templates

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

---

## `astro.config.ts`

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

## `tsconfig.json`

```json
{
  "extends": "astro/tsconfigs/strict",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": { "@/*": ["src/*"] }
  }
}
```

## `package.json` additions

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

## Husky setup

```bash
bunx husky init
echo "bunx lint-staged --allow-empty" > .husky/pre-commit
echo "bun check" >> .husky/pre-commit
```

## `eslint.config.ts`

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

## `prettier.config.ts`

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

## `.gitignore`

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

## `src/styles/app.css`

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

## `src/libs/cn.ts`

```typescript
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export const cn = (...inputs: ClassValue[]) => twMerge(clsx(inputs))
```

## `src/layouts/base.astro`

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

## `src/components/layout/base-head.astro`

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

## `src/components/layout/theme-toggle.astro`

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

## `src/pages/robots.txt.ts`

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

## `public/_headers`

```
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=()
  Strict-Transport-Security: max-age=31536000; includeSubDomains
```

## `public/_redirects`

```
# Old URL -> New URL (301 permanent)
# /old-path /new-path 301
```

Compare original URLs vs clone URLs. Add 301s for any that changed. Handle trailing slash consistency -- set `trailingSlash` in astro.config to match original site's behavior.
