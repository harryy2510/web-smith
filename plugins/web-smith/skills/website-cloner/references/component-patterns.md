# Component Patterns

## Content Strategy

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

---

## Component Rules

**All `.astro` components** (zero JS). Only exceptions:
- Theme toggle (`<script is:inline>`)
- Mobile menu toggle (`<script is:inline>`)
- FAQ accordion (native `<details>`/`<summary>`)

**Naming:** kebab-case always. `hero-section.astro`, `nav.astro`, `page-hero.astro`.

---

## Mobile Menu Toggle

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

---

## Active Nav Link

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

---

## CSS-only Dropdown Nav (no JS)

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

---

## Logo Dark/Light Variant

```astro
<img src="/images/logos/logo.svg" alt="Logo" class="h-8 dark:hidden" />
<img src="/images/logos/logo-white.svg" alt="Logo" class="hidden h-8 dark:block" />
```

---

## Third-party Embeds

| Embed | Approach |
|---|---|
| YouTube/Vimeo | `lite-youtube`/`lite-vimeo` web component (lazy, tiny JS) |
| Google Maps | Static map image linking to Google Maps, or `<iframe loading="lazy">` |
| Calendly/booking | `<iframe loading="lazy">` |
| Social feeds | Static snapshot, link to profile |
| Chat widgets | Load on interaction (click-to-load), not page load |
| Analytics | `@astrojs/partytown` to offload to web worker |

---

## Forms

Options for form backends (static site has no server):
1. **Formspree** / **Formspark** -- simple, hosted form endpoint
2. **Cloudflare Worker** -- custom endpoint if complex logic needed
3. **mailto: link** -- simplest, no backend needed
4. At minimum: form UI renders correctly, submit shows a thank-you state. Never broken/erroring forms.

---

## 404 Page

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

---

## Animations and Carousels

**No animation libraries.** Use native APIs and CSS only:

| Pattern | Approach |
|---|---|
| Carousel/slider | CSS `scroll-snap-type: x mandatory` + `scroll-snap-align: start`. Native scroll. Zero JS. |
| Auto-advancing carousel | CSS `@keyframes` with `animation` to scroll. Or minimal `<script is:inline>` with `setInterval` + `scrollTo`. |
| Fade in on scroll | CSS `@starting-style` + `transition` (modern). Or `IntersectionObserver` in `<script is:inline>` adding a class. |
| Hover effects | Tailwind `transition-*` + `hover:` utilities. |
| Accordion | Native `<details>`/`<summary>`. Zero JS. |
| Tabs | CSS `:target` selector or radio-button technique. Or minimal inline JS toggling `hidden`. |
| Smooth scroll | CSS `scroll-behavior: smooth` (already in `app.css`). |
| Progress bars | CSS `@keyframes` or `width` transition. |
| Parallax | `background-attachment: fixed`. No JS parallax libraries. |
| Video background | `<video autoplay muted loop playsinline poster="frame.jpg">`. Static poster as fallback. |
| Lottie animations | Replace with static frame or CSS equivalent. If critical: inline the lottie-player as an exception to zero-JS rule, document in CLONE-WIKI.md. |

Always respect `prefers-reduced-motion`: disable or reduce all animations.

---

## Internationalization (i18n)

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
