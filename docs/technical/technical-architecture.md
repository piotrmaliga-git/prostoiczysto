# Technical architecture

This document defines implementation constraints and cross-cutting quality requirements. Visitor-facing behavior is specified in [Product requirements](../product/product-requirements.md).

## Stack and engineering constraints

- Runtime: Node.js `24.18.0` (current environment).
- Package manager: npm `11.16.0`; use `npm install` and npm scripts.
- Latest stable Angular; fully standalone components.
- Zoneless change detection and `ChangeDetectionStrategy.OnPush` on every component.
- Use Angular's new control flow (`@if`, `@for`, `@switch`).
- Angular SSR with prerendering of `/pl` and `/en`.
- Angular hydration **and incremental hydration are required**.
- Lazy-loaded routes.
- Latest Tailwind CSS plus CSS variables for theming.
- Transloco for i18n.
- Vitest for unit tests; use Angular's default Vitest runner and add focused component/service tests for application behavior.
- Follow Angular conventions and the architecture diagram at `../images/architektura.png`. The diagram wins for folder structure and layering; this specification wins for behavior.
- Use **oxlint only**. Do not install or configure ESLint or Prettier. Add `npm run lint` to run oxlint.
- Guard browser-only API calls (`localStorage`, `window`, `document`, `matchMedia`, `IntersectionObserver`) with `isPlatformBrowser` or `afterNextRender`, so SSR/prerender does not break.
- Do not use `ng new` interactive prompts; pass all flags explicitly for a reproducible scaffold.

## Hydration

### Goal

The prerendered HTML for `/pl` and `/en` must be reused by the browser without DOM recreation, flicker, or layout shift. Use incremental hydration only for components that provide substantial below-the-fold interactivity or downloadable JavaScript; do not defer static sections merely because they appear below the fold.

Read before Phase 1:

- <https://angular.dev/guide/hydration>
- <https://angular.dev/guide/incremental-hydration>

### Setup and stability

- Configure `provideClientHydration()` in both client app config and server bootstrap config; verify the Angular CLI SSR scaffold does both.
- Incremental hydration and event replay are included by default with `provideClientHydration()`. Do not add `withNoIncrementalHydration()` or a separate `withEventReplay()`.
- Clicks made before hydration finishes are replayed automatically.
- Keep `preserveWhitespaces` at its default (`false`) in browser and server tsconfig files.
- Zoneless is the change-detection mode. Hydration waits for application stability, so avoid startup `setInterval` / `setTimeout`, unresolved promises, and pending microtasks. Use CSS, not JS timers, for continuous animation.
- Set up scroll reveal and other browser-only behavior inside `afterNextRender`. Keep content visible by default if JavaScript or the observer is unavailable.

### Incremental hydration plan

The landing page is mostly static content. Keep it eager unless a section has a meaningful hydration cost or interactive behavior that justifies deferral.

| Section | Strategy |
|---|---|
| Header + hero | Eager; no `@defer` |
| About | Eager; no `@defer` |
| Machines | Eager; no `@defer` |
| Gallery + lightbox | Eager; if a lightbox is implemented, defer only the optional lightbox overlay and hydrate it on its first user interaction |
| Pricing | Eager; no `@defer` |
| FAQ | Eager; native `<details>`/`<summary>` requires no JavaScript |
| Contact form | Eager; no `@defer` |
| Footer | Eager; no `@defer` |

Rules:

- Keep section and component deferrals minimal. In the initial implementation, use no whole-section deferral.
- If the optional lightbox is implemented, defer only the lightbox overlay, not the Gallery section or its images.
- Every `@defer` block needs a minimal `@placeholder`; do not reserve a large layout box for an absent lightbox or modal.
- Keep every `<section id="...">` wrapper outside a `@defer` block so navigation targets and section content remain server-rendered.
- Do not use nested `@defer` blocks.
- Do not use `hydrate when` or `hydrate never` unless a later, documented requirement introduces them.
- Nothing outside an optional lightbox may depend on it. Navigation, back-to-top, theme, and language switching stay outside.
- If the Gallery lightbox remains optional and does not require a separate bundle, keep the Gallery and lightbox eager.

### Hydration-safe coding

- No direct DOM manipulation in components: no `innerHTML`/`outerHTML`, `appendChild`, `document.querySelector` mutations, or node movement. Use Angular templates, bindings, and `Renderer2`.
- Modal and lightbox render through Angular (for example `@if` or native `<dialog>`); do not append nodes to `document.body` manually.
- Server and client must produce the same initial output. Do not branch templates with `@if (isPlatformBrowser(...))`.
- Browser-derived state (theme, scroll position, hover) may change only after hydration and must not change the first render.
- Theme toggle: render both moon and sun icons; CSS switches visibility using the `<html>` `data-theme` attribute set by the inline no-flash script. Unit tests assert service state and `aria-pressed` / `data-theme`.
- Language comes from the URL, so server and client render the same text.
- Use valid HTML: no `<div>` inside `<p>`, no nested links, and explicit `<tbody>` in tables.
- The inline theme script may modify attributes on `<html>` only, never content inside the Angular root.
- DOM-touching third-party scripts/libraries (analytics, widgets) load only after hydration via `afterNextRender`. Prefer no third-party scripts.
- `ngSkipHydration` is a last resort; target zero uses. Document any use and how to remove it in the README.
- Set head tags (title, meta, canonical, hreflang, JSON-LD) through Angular `Title` / `Meta` services or an SEO service during SSR, not template `innerHTML`.

### i18n and hydration

- Load Transloco translations before first render on both server and client so prerendered text matches and translation keys do not flash.
- Use a loader that works during prerender. Rely on Angular HTTP transfer cache (included with hydration) so the client does not refetch server-loaded translations.
- `withI18nSupport()` is only for Angular built-in `i18n` template blocks. This project uses Transloco: do not use `i18n` attributes or add `withI18nSupport()`.

## GitHub Actions CI/CD

GitHub Actions workflows must use the pinned Node.js `24.18.0` and npm `11.16.0` versions. Use `npm ci`, cache the npm dependency cache, and fail on any warning, error, or failed assertion relevant to the pipeline.

### Pull requests

Run these checks sequentially for pull requests targeting the main branch. A failed gate stops the pipeline immediately; the next gate starts only after the previous gate passes:

1. **Build**
   - Install dependencies and produce a complete Angular SSR/prerender build.
   - Stop on dependency installation, compilation, prerendering, or build failure.

2. **Lint and format**
   - Run `npm run lint` and the repository's format-check script.
   - Stop on lint or formatting failure.
   - Do not install or configure ESLint or Prettier.

3. **Check translations**
   - Validate that Polish and English have the same Transloco keys.
   - Stop if values are missing or placeholder-key references remain unresolved.

4. **Unit tests**
   - Run the focused Vitest suite for application behavior.
   - Stop on any failed test or test-run error.

5. **E2E tests**
   - Start or use the verified build and run Playwright for `/pl` and `/en`.
   - Stop on a failed user-flow test, console error, or hydration warning.

Run every gate against the same source revision and dependency lockfile. Upload build, test, and Playwright reports when a gate fails.

### Merge to main

For merges to the main branch, run these gates sequentially. A failed gate stops the pipeline immediately:

1. **Build**
   - Reproduce the production SSR/prerender build.
   - Stop on any build failure.

2. **Unit tests**
   - Run the focused Vitest suite.
   - Stop on any failed test or test-run error.

3. **E2E tests**
   - Run the Playwright suite against the production build.
   - Stop on a failed user-flow test, console error, or hydration warning.

Upload build and test artifacts when a gate fails. Do not deploy automatically unless a separate deployment workflow and hosting configuration are added and approved.

### Hydration verification

- In dev mode, verify hydration stats in the browser console on `/pl` and `/en`.
- Confirm prerendered HTML contains hydration markers (for example `ngh` attributes).
- Check for no hydration mismatch and no “Application remains unstable” (NG0506) errors.
- Use Angular DevTools hydration overlay once to verify any deferred interactive island stays dehydrated until its trigger, then hydrates without visual jumps.
- Playwright must fail on console errors or hydration warnings.

### Acceptance tests

Vitest coverage targets are not numerical. Test key behavior and critical regressions instead.

Unit and component tests must cover:

- Polish and English route rendering with the correct page language.
- Root language resolution using saved preference, browser language, and Polish fallback.
- Language switching that preserves the current section hash and updates `localStorage` and `<html lang>`.
- Theme defaults and persisted preferences, including `aria-pressed` and the theme attribute.
- Header navigation and mobile menu behavior, including link activation and Escape close.
- Pricing cards, machine cards, external machine links, and placeholder rendering.
- Gallery image alternatives and optional lightbox keyboard behavior.
- FAQ native accordion behavior.
- Contact form required fields, invalid values, loading state, success, and error notifications.
- Back-to-top visibility and keyboard activation.
- 404 route localization and return-home navigation.

Browser tests must verify:

- `/pl` and `/en` render the correct content and metadata.
- `/` redirects according to the saved preference or browser language.
- Navigation smooth-scroll targets are reachable and section IDs exist.
- The mobile menu opens, closes, and remains keyboard operable.
- Theme and language preferences survive reload.
- The contact form displays validation and submission states without leaving an invalid submitted state behind.
- Key sections are present in prerendered HTML.
- The site has no hydration mismatch, console warning, or application stability error.
- Keyboard focus and visible focus states work for navigation, FAQ, modal/lightbox, and form controls.
- WCAG AA contrast and reduced-motion behavior are visually checked in dark and light themes.

## SEO

- Semantic `header`, `nav`, `main`, `section`, and `footer`; one `h1`; logical heading order.
- Per-language title and meta description, Open Graph and Twitter tags.
- Canonical URL per language plus `hreflang` (`pl`, `en`, `x-default`) on both pages.
- JSON-LD `LocalBusiness` with placeholder address/contact details; optional `FAQPage` built from FAQ data. Markup must be valid; FAQ rich results are not guaranteed.
- `sitemap.xml` for `/pl` and `/en` with alternates, `robots.txt`, and favicon.
- All section content, including deferred sections, must be present in prerendered HTML.

## Accessibility

- Skip-to-content link and visible focus states.
- Menu ARIA (`aria-expanded`, `aria-controls`); toast roles (`role="status"` / `role="alert"`); form errors announced and associated with fields.
- Form fields have labels and `aria-describedby` for errors.
- Full keyboard navigation and `prefers-reduced-motion` support.
- Scroll-triggered animations respect reduced motion and do not interfere with navigation, hydration, or access to section content.
- Translate alt text for all images, including Machines.
- FAQ uses real `<summary>` controls, operable with Enter/Space and with visible focus.
