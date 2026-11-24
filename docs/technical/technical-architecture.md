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

The prerendered HTML for `/pl` and `/en` must be reused by the browser without DOM recreation, flicker, or layout shift. Below-the-fold sections should hydrate lazily to keep initial JavaScript small.

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

Header, hero, and About are eager. Keep the back-to-top dependency outside deferred sections.

| Section | Strategy |
|---|---|
| Header + hero | Eager; no `@defer` |
| About | Eager; no `@defer` |
| Machines | `@defer (hydrate on viewport)` |
| Gallery + lightbox | `@defer (hydrate on viewport; hydrate on interaction)` |
| Pricing | `@defer (hydrate on viewport)` |
| FAQ | `@defer (hydrate on viewport; hydrate on interaction)` |
| Contact form | `@defer (hydrate on viewport; hydrate on interaction)` |
| Footer | `@defer (hydrate on viewport)`; `hydrate never` is allowed only if it contains nothing but plain external `<a href>` links |

Rules:

- Every `@defer` block needs a `@placeholder`, including for client-side navigation. Size it to roughly match the real section.
- Keep each `<section id="...">` wrapper outside its `@defer` block so navigation targets exist. Keep headings and static copy server-rendered for SEO.
- Do not use `hydrate when`; it only works for the top-most dehydrated block.
- Avoid nested `@defer` blocks unless there is a reason. `hydrate never` blocks its whole nested subtree.
- Nothing outside a deferred section may depend on a component inside it. Navigation, back-to-top, theme, and language switchers stay outside.

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

### Hydration verification

- In dev mode, verify hydration stats in the browser console on `/pl` and `/en`.
- Confirm prerendered HTML contains hydration markers (for example `ngh` attributes).
- Check for no hydration mismatch and no “Application remains unstable” (NG0506) errors.
- Use Angular DevTools hydration overlay once to verify deferred sections stay dehydrated until their trigger, then hydrate without visual jumps.
- Playwright must fail on console errors or hydration warnings.

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
