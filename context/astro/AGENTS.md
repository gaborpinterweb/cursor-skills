# AGENTS.md — Astro Website Repository Rules

## Mandatory Workflow
1. **NEVER** run `git commit`, `git push`, or deploy commands without explicit human approval.
2. **ALWAYS** create an implementation plan and get developer approval before making changes.
3. **NEVER** modify files outside the scope of the approved plan.
4. **Updating Rules**: If you need to implement any new rules that would be better to follow for future changes, you must add them to this AGENTS.md file. However, **before adding new rules, you must ask the user for explicit approval.**

## Code Standards
### HTML
- Strict W3C WCAG 2.1 AA compliance on every page/component.
- Semantic HTML5 elements: `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<footer>`, `<aside>`.
- Single `<h1>` per page. Heading hierarchy must never skip levels (h1 → h2 → h3).
- All interactive elements must have unique, descriptive `id` attributes.
- All images must have meaningful `alt` text (or `role="presentation" aria-hidden="true"` for decorative).
- ARIA landmarks and labels on all navigation regions.
- Skip-to-content link on every page.

### CSS/SCSS
- BEM naming convention for all classes (including classes used with scoped styles).
- Prefer Astro scoped `<style>` in components; put shared tokens and resets in global styles (e.g. `src/styles/`).
- **Mobile-First Design**: Mobile-first breakpoints only (`min-width` media queries).
- **Dark Mode Default**: Use dark mode as the default theme. Maintain the same pattern and consistency throughout the site, even when making changes.
- **CSS Consistency**:
  - Follow the exact same pattern for dark mode and light mode designs, with dark mode as the primary approach (dark mode first).
  - Use variables for colors and font sizes.
  - Before defining new variables, first check the variables list (name and value) to see if a suitable one exists.
  - Do not reuse a variable that was made for a specific semantic purpose just because the values match.
  - If a new variable is needed for a new/non-existing purpose, create it and add comments explaining its purpose.
  - Write comments like a human developer. Do NOT use emojis or icons (e.g., checkmarks, planes) in comments.
- **Measurements from Design**: Each and every measurement must come directly from the Axure design. Do not invent or use AI-generated values for measurements. Instead, check the design or explicitly ask the user/developer to provide the required measurements.
- All sizing in `rem` via a shared `px-to-rem()` helper (or equivalent token map; base 16px).
- All colors, font sizes, spacing in CSS custom properties (`:root` / design-token files).
- No external CSS frameworks (no Bootstrap, Tailwind, etc.).
- **DOM Depth & Nesting**: Avoid excessive DOM element depth. Write clean, flat HTML and CSS to adhere to best practices.

### JavaScript
- Default to zero client JS. Prefer `.astro` components that ship HTML/CSS only.
- Lightweight vanilla JS only when interactivity is required. No external libraries (no jQuery, no React/Vue/Svelte islands) unless explicitly approved in the plan.
- Keep client scripts small and colocated (e.g. `<script>` in the component that needs them, or `src/scripts/`). Prefer `is:inline` only when necessary; otherwise let Astro/Vite bundle.
- Re-initialize behavior on client navigations when View Transitions are enabled (`astro:page-load` / `astro:after-swap`).
- No `eval()`, `document.write()`, or other unsafe patterns.
- No unnecessary `client:*` directives. If an island is ever approved, choose the least eager directive (`client:visible` / `client:idle` over `client:load`).

### Assets & GDPR
- All assets (fonts, images, icons, scripts) MUST be self-hosted.
- Zero external CDN references (no googleapis.com, cdnjs, unpkg, jsDelivr, etc.).
- Fonts as self-hosted woff2 under `public/fonts/` (or the repo’s established fonts path), with preload in the base layout `<head>`.
- Icons as inline SVG via a shared component (e.g. `InlineSvg.astro`), not external icon CDNs.
- Put optimizable images in `src/assets/` and use Astro’s `<Image />` / `<Picture />` from `astro:assets`. Use `public/` only for files that must keep a stable raw URL (favicons, `robots.txt`, etc.).

### Astro Tooling
- **Static output by default**: `output: 'static'` unless the approved plan explicitly requires SSR/hybrid.
- Set `site` in `astro.config` for correct canonicals and sitemap generation.
- Use Content Collections with Zod schemas (`src/content.config.ts`) for structured content; fail the build on invalid frontmatter.
- Use file-based routing under `src/pages/`. Prefer shared layouts in `src/layouts/` and reusable UI in `src/components/`.
- Do not relocate Astro defaults (`src/pages`, `src/content`, `public`) without an approved plan.
- Add integrations only when required (`@astrojs/sitemap`, `@astrojs/mdx`, etc.). Do not add Tailwind or UI kits.
- Prefer native Sass/SCSS via Vite when needed; keep tokens in CSS custom properties either way.

### Performance
- Ship minimal JS. No framework runtime on pages that do not need it.
- Images via the shared image component / `astro:assets` (modern formats, explicit dimensions, aspect-ratio wrapper for CLS=0).
- Font preloading in the base layout `<head>`.
- `loading="lazy"` on all non-hero images; `loading="eager"` (and appropriate `fetchpriority`) only for above-fold hero.
- Enable sitemap generation (`@astrojs/sitemap` or equivalent) from `site`.
- If View Transitions / `<ClientRouter />` are used, honor `prefers-reduced-motion` and avoid persisting large stateful subtrees.

### SEO
- Every page: `<title>`, `<meta description>`, `<link rel="canonical">`, hreflang alternates when multilingual.
- JSON-LD WebPage schema on every page (centralize in the base layout or an `Seo.astro` component).
- Sitemap generated at build time.
- `<meta name="robots">` from site config / content fields.

### Git Workflow
- NEVER push directly. All changes via pull request.
- NEVER auto-commit generated files (`dist/`, `.astro/`, etc.).
- Review all changes before committing.
