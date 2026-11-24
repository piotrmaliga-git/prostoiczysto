# Technical requirements

This document defines the application's technical constraints and cross-cutting requirements. Project structure and responsibility boundaries are documented separately in [Project architecture](./architecture.md). Key test behaviors are documented separately in [Testing](./testing.md). Visitor-facing behavior is specified in [Product requirements](../product/product-requirements.md).

## Stack and engineering constraints

- At scaffold time, use the latest stable Node.js LTS release and latest stable compatible npm release. Record the exact versions in the repository's runtime/package-manager configuration and use those same versions in CI.
- Use npm and npm scripts. Commit the lockfile and use `npm ci` in CI.
- For libraries and tooling without a specified version, select the latest stable release compatible with the rest of the stack at scaffold time. Pin resolved dependency versions in `package.json` and `package-lock.json`; CI must use the lockfile.
- Use the latest stable Angular at scaffold time, with fully standalone components.
- Use zoneless change detection and `ChangeDetectionStrategy.OnPush` on every component.
- Use Angular's new control flow (`@if`, `@for`, `@switch`).
- Use Angular SSR with prerendering of `/pl` and `/en`.
- Angular hydration is required. Use incremental hydration selectively, as described below.
- Use lazy-loaded routes.
- Use the latest stable Tailwind CSS at scaffold time plus CSS variables for theming.
- Use Transloco for i18n.
- Use Vitest for unit/component tests and Playwright for E2E tests; see [Testing](./testing.md).
- Follow Angular conventions and [Project architecture](./architecture.md). The older architecture image at `../images/architektura.png` is only a visual reference and is not an implementation authority.
- Use Oxlint for linting and Oxfmt for formatting. They are separate Oxc tools: `npm run lint` runs `oxlint`, and `npm run format:check` runs `oxfmt --check`. Do not install or configure ESLint or Prettier.
- Guard browser-only API calls (`localStorage`, `window`, `document`, `matchMedia`, `IntersectionObserver`) with `isPlatformBrowser` or `afterNextRender`, so SSR/prerender does not break.
- Do not use `ng new` interactive prompts; pass all flags explicitly for a reproducible scaffold.

## Architecture and responsibility boundaries

- Keep page sections, shared UI, route configuration, services, pipes, utilities, localization, and styles in the layers defined in [Project architecture](./architecture.md).
- Keep product facts and visitor-facing requirements in [Product requirements](../product/product-requirements.md); keep technical implementation constraints in this document.
- Keep uncertain business details as clearly marked placeholders. Do not invent prices, contact details, marketing copy, or other business facts.
- Prefer simple, maintainable Angular patterns and avoid introducing abstractions without a demonstrated need.

## SSR and hydration

The prerendered HTML for `/pl` and `/en` must be reused by the browser without DOM recreation, flicker, or layout shift. Use incremental hydration only for components that provide substantial interactivity or downloadable JavaScript; do not defer static sections merely because they appear below the fold.

Read before implementation:

- <https://angular.dev/guide/hydration>
- <https://angular.dev/guide/incremental-hydration>

### Setup and stability

- Configure `provideClientHydration()` in the client application and verify SSR configuration against the selected Angular version.
- Use Angular's documented defaults for incremental hydration and event replay; do not add redundant providers or opt-outs.
- Keep `preserveWhitespaces` at its default (`false`) in browser and server TypeScript configuration.
- Hydration waits for application stability. Avoid startup timers, unresolved promises, and pending microtasks. Use CSS, not JavaScript timers, for continuous animation.
- Set up scroll reveal and other browser-only behavior inside `afterNextRender`. Keep content visible by default if JavaScript or the observer is unavailable.

### Incremental hydration

The landing page is mostly static. Keep sections eager unless a specific interactive component has a meaningful hydration cost that justifies deferral.

| Section/component | Strategy |
|---|---|
| Header and hero | Eager |
| About | Eager |
| Machines | Eager |
| Gallery | Eager |
| Optional lightbox overlay | May be deferred and hydrated on first interaction if its separate bundle cost justifies it |
| Pricing | Eager |
| FAQ | Eager; native `<details>`/`<summary>` needs no JavaScript |
| Contact form | Eager |
| Footer | Eager |

Rules:

- Do not defer whole sections in the initial implementation.
- If the optional lightbox is implemented, defer only its overlay, not the Gallery section or its images.
- Every `@defer` block needs a minimal `@placeholder`.
- Keep every `<section id="...">` wrapper outside a `@defer` block so navigation targets and section content remain server-rendered.
- Do not use nested `@defer`, `hydrate when`, or `hydrate never` unless a later requirement justifies them.
- Nothing outside an optional lightbox may depend on it. Navigation, back-to-top, theme, and language switching stay outside.

### Hydration-safe implementation

- No direct DOM mutation in components: avoid `innerHTML`/`outerHTML`, `appendChild`, `document.querySelector` mutations, or node movement. Use Angular templates, bindings, and `Renderer2`.
- Modal and lightbox content must render through Angular (for example `@if` or native `<dialog>`); do not append nodes to `document.body` manually.
- Server and client must produce the same initial output. Do not branch templates on browser-only state.
- Browser-derived state (theme, scroll position, hover) may change only after hydration and must not change the first render.
- Render both theme icons; CSS switches visibility using the `<html>` `data-theme` attribute set by the inline no-flash script.
- Language comes from the URL, so server and client render the same text.
- Use valid HTML: no `<div>` inside `<p>`, no nested links, and explicit `<tbody>` in tables.
- The inline theme script may modify attributes on `<html>` only, never content inside the Angular root.
- DOM-touching third-party scripts load only after hydration via `afterNextRender`. Prefer no third-party scripts.
- `ngSkipHydration` is a last resort; target zero uses. Document any use and how to remove it in the README.
- Set title, meta, canonical, hreflang, and JSON-LD through Angular `Title`/`Meta` services or an SEO service during SSR, not template `innerHTML`.

### Internationalization and hydration

- Load Transloco translations before first render on both server and client so prerendered text matches and translation keys do not flash.
- Use a loader that works during prerender. Rely on Angular HTTP transfer cache so the client does not refetch server-loaded translations.
- `withI18nSupport()` is only for Angular built-in `i18n` template blocks. This project uses Transloco: do not use `i18n` attributes or add `withI18nSupport()`.

## GitHub Actions CI/CD

GitHub Actions workflows must use the exact Node.js and npm versions recorded in the repository runtime/package-manager configuration. Use `npm ci`, cache the npm dependency cache, and fail on warnings, errors, or failed assertions relevant to the pipeline.

Run gates sequentially: a gate starts only after the previous gate passes, and any failure stops the pipeline.

### Commit message linting

Use Commitlint with `@commitlint/config-conventional` and this configuration:

```js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',
        'fix',
        'docs',
        'style',
        'refactor',
        'perf',
        'test',
        'build',
        'ci',
        'chore',
        'revert',
      ],
    ],
    'header-max-length': [2, 'always', 200],
  },
};
```

When the application and package manifest are scaffolded, add Commitlint and `@commitlint/config-conventional` as development dependencies. Configure Commitlint as a local Git `commit-msg` hook so it validates each commit message before the commit is created. This is a local commit-time check, not a separate pull-request CI gate.

### Pull requests

Run these checks sequentially for pull requests targeting the main branch:

1. **Build** — install dependencies and produce a complete Angular SSR/prerender build. Stop on dependency installation, compilation, prerendering, or build failure.
2. **Lint and format** — run `npm run lint` and `npm run format:check` (Oxlint and Oxfmt). Do not install or configure ESLint or Prettier.
3. **Check translations** — validate that Polish and English have the same Transloco keys. Stop if values are missing or placeholder-key references remain unresolved.
4. **Unit tests** — run the focused Vitest suite for application behavior.
5. **E2E tests** — run Playwright for `/pl` and `/en`; fail on a failed user-flow test, console error, or hydration warning.

Run every gate against the same source revision and dependency lockfile. Upload build, test, and Playwright reports when a gate fails.

### Merge to main

Run these checks sequentially on merges to the main branch:

1. **Build** — reproduce the production SSR/prerender build.
2. **Unit tests** — run the focused Vitest suite.
3. **E2E tests** — run Playwright against the production build and fail on a failed user-flow test, console error, or hydration warning.

Upload build and test artifacts when a gate fails. Do not deploy automatically unless a separate deployment workflow and hosting configuration are added and approved.

## SEO

- Planned production canonical origin: `https://www.prostoiczysto.pl`. Domain registration, DNS configuration, and hosting setup remain prerequisites to launch.
- Use `https://prostoiczysto.pl` as the non-canonical host and redirect it to `https://www.prostoiczysto.pl`. Redirect HTTP requests to HTTPS for both hostnames.
- Configure a valid TLS certificate covering both `www.prostoiczysto.pl` and `prostoiczysto.pl`; do not launch without working HTTPS on both names, including the apex-to-www redirect.
- Use the planned canonical origin consistently in canonical URLs, `hreflang`, sitemap, Open Graph metadata, and JSON-LD. Keep the hosting provider and its DNS records unspecified until deployment hosting is selected.
- Use semantic `header`, `nav`, `main`, `section`, and `footer`; one `h1`; and a logical heading order.
- Provide per-language title and meta description, Open Graph, and Twitter tags.
- Provide canonical URLs per language plus `hreflang` (`pl`, `en`, `x-default`) on both pages, using the planned production origin above.
- Add `LocalBusiness` JSON-LD with placeholder address/contact details. `FAQPage` from FAQ data is optional; markup must be valid, and FAQ rich results are not guaranteed.
- Provide `sitemap.xml` for `/pl` and `/en` with alternates, `robots.txt`, and favicon.
- All section content, including content in any deferred interactive island, must be present in prerendered HTML.

## Accessibility

- Provide a skip-to-content link and visible focus states.
- Set menu ARIA (`aria-expanded`, `aria-controls`); use toast roles (`role="status"` / `role="alert"`); announce form errors and associate them with fields.
- Form fields need labels and `aria-describedby` for errors.
- Support full keyboard navigation and `prefers-reduced-motion`.
- Scroll-triggered animations must respect reduced motion and not interfere with navigation, hydration, or access to section content.
- Translate alt text for all images, including Machines.
- FAQ uses real `<summary>` controls, operable with Enter/Space and with visible focus.

See [Testing](./testing.md) for acceptance checks covering SEO, accessibility, and hydration behavior.
