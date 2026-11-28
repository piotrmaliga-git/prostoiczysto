# Project architecture

This document is the authoritative description of the repository structure, Angular application layers, responsibility boundaries, and dependencies.

Technical implementation constraints and runtime behavior are specified in [Technical requirements](./technical-requirements.md). Product behavior is specified in [Product requirements](../product/product-requirements.md).

## Textual project structure

The repository root contains orchestration documentation, GitHub configuration, Git hooks, and the Angular application. The Angular application is scaffolded directly at the repository root, next to `.github/`, `.husky/`, and `orchestration/`, so that Husky, GitHub workflows, and tooling share one root. Do not create an application subdirectory. Application paths below are relative to the repository root.

```text
repository-root/
├── .github/
│   ├── agents/
│   │   ├── orchestrator.agent.md
│   │   └── executor.agent.md
│   ├── workflows/
│   └── PULL_REQUEST_TEMPLATE.md
├── orchestration/
│   └── docs/
│       ├── product/
│       │   ├── product-requirements.md
│       │   └── design-and-assets.md
│       ├── technical/
│       │   ├── architecture.md
│       │   ├── technical-requirements.md
│       │   └── testing.md
│       └── images/
├── .husky/
│   ├── pre-commit
│   └── commit-msg
├── e2e/
│   └── [Playwright tests and fixtures]
├── public/
│   ├── favicon.ico or favicon.svg
│   ├── robots.txt
│   └── sitemap.xml
├── src/
│   ├── app/
│   │   ├── pages/
│   │   ├── components/
│   │   ├── shared/
│   │   │   └── ui/
│   │   ├── routes/
│   │   ├── service/
│   │   ├── pipes/
│   │   └── utils/
│   │       ├── constants/
│   │       ├── enums/
│   │       ├── interfaces/
│   │       ├── translations/
│   │       └── types/
│   ├── assets/
│   │   ├── images/
│   │   └── fonts/
│   ├── locale/
│   │   ├── messages.xlf
│   │   └── messages.pl.xlf
│   ├── environments/
│   │   ├── environment.ts
│   │   └── environment.development.ts
│   ├── styles/
│   ├── index.html
│   ├── main.ts
│   ├── main.server.ts
│   └── server.ts
├── .gitignore
├── .nvmrc
├── README.md
├── angular.json
├── package.json
├── package-lock.json
├── commitlint.config.js
├── tsconfig.json
├── tsconfig.app.json
├── tsconfig.spec.json
├── vitest.config.ts
├── playwright.config.ts
└── vercel.json
```

The exact scaffold may place SSR and server configuration files in additional Angular-generated locations. Do not move or rename the application layers above unless a task explicitly updates this architecture. GitHub workflows run application commands from the repository root.

## Folder responsibilities

### `src/app/pages/`

- Contains language-level route views and route composition.
- `/pl` and `/en` are landing-page views.
- `/` is the browser-language redirect route.
- `**` is the lazy-loaded, localized 404 route.
- A page composes sections and route-level services.
- A page must not own the reusable implementation of a section or shared control.

### `src/app/components/`

- Contains landing-page sections and feature-specific components.
- Intended sections are Header, Hero, About, Machines, Gallery, Pricing, FAQ, Contact, and Footer.
- Components may depend on shared UI, services, pipes, and utilities.
- Components must not depend on another page.

### `src/app/shared/ui/`

- Contains reusable, standalone, OnPush presentation controls with no product-specific copy.
- Required controls are Button, Modal, Toast/Notification, Card, SectionHeading, FormField, Icon, and Accordion.
- Components must support `data-testid`, accessible defaults, and hydration-safe rendering.

### `src/app/routes/`

- Contains route definitions, route guards or resolvers, and lazy-loading configuration.
- Contains route-level metadata and redirects for `/`, `/pl`, `/en`, and `**`.

### `src/app/service/`

- Contains injectable application services.
- Intended responsibilities include notifications, theme, SEO/metadata, and contact submission. Locale is selected by the localized build and URL; do not add a service that swaps translation catalogs at runtime.
- Services must not contain standalone page markup or product copy.

### `src/app/pipes/`

- Contains pure presentation transforms only.

### `src/app/utils/`

- Contains shared, framework-independent utilities with no DOM or injectable-service dependency. Organize shared code into focused category folders:
  - `constants/` for reusable constant values.
  - `enums/` for runtime enum declarations.
  - `interfaces/` for shared object contracts.
  - `translations/` for code-based localization helpers and messages.
  - `types/` for type aliases, unions, and other shared type declarations.
- Keep each concern in a descriptive, dedicated file. Do not put reusable declarations or translation cases in component or service implementation files.
- Use these folders for shared concerns; keep feature-specific declarations in separate files within their feature area rather than turning `utils/` into a catch-all.

### `src/app/utils/translations/`

- Contains pure translation helpers, localized constants, and locale-specific mappings needed by TypeScript code.
- Keep `$localize` messages and translation-selection logic out of component logic; components should consume helpers from this folder when they need code-generated localized text.
- Keep helpers free of component state, DOM access, and injectable services.
- Static template copy stays in templates marked with Angular `i18n`; do not move it into utilities just to translate it.

### `src/assets/`

- Contains bundled application assets such as responsive copies of placeholder images and local font files.
- Source assets remain in `orchestration/docs/images/` and must not be edited in place as an implementation step.

### `src/locale/`

- Contains the catalogs for Angular's built-in i18n.
- English is the source locale. Extract marked template messages and `$localize` strings into `messages.xlf`; maintain Polish translations in `messages.pl.xlf`.
- Keep source code, component names, service names, developer comments, technical identifiers, and test descriptions in English.
- Configure Angular's localized builds to emit the English source locale at `/en` and the Polish translation at `/pl`.
- Validate the Polish XLIFF catalog against the extracted English messages; do not maintain a parallel JSON dictionary or duplicate translated copy in TypeScript constants.
- Compile locale-specific messages into each output so server rendering and hydration use the same locale without a runtime translation loader.

### `src/environments/`

- Contains build environment configuration.
- `environment.ts` is the production environment configuration.
- `environment.development.ts` is the development environment override.
- The Formspree endpoint belongs in environment configuration and must not be hard-coded in a component or service.

### `src/styles/`

- Contains global styles, Tailwind entry styles, theme variables, and reset/base styles.
- Components keep component-local styles unless a rule is intentionally global.

### `public/`

- Contains files that must be served unchanged at the site root, including `robots.txt`, sitemap, and favicon.

### `e2e/`

- Contains Playwright browser tests, page helpers, and test fixtures.

## Cross-layer dependency direction

- Pages depend on components and shared UI.
- Components and pages may use services, pipes, and utilities.
- Shared UI must not import from pages or feature components.
- Services must not import shared UI or page components unless a service is explicitly browser-behavior-specific and otherwise remains decoupled.
- Pipes and utilities must remain independent of page and component state.
- Keep nonlocalized product facts and machine data in one constants/data source. Mark visitor-facing copy with Angular i18n in templates or `$localize` in TypeScript, and do not duplicate it across components.
- XLIFF translation catalogs contain public translated messages only; never put environment secrets or deployment configuration in them.

## Application state and configuration

- Language is route-derived from `/pl` or `/en`; it must not be inferred globally outside the root redirect.
- Theme is initialized by the inline script and then managed by a theme service.
- Root `/` language resolution may read `localStorage` and `navigator.language`, but SSR and prerender must not assume those browser APIs exist.
- The language switch and theme toggle are the only visitor preferences that require persistence; both must have safe fallbacks.
- The Formspree endpoint belongs in environment configuration and is never hard-coded in a component.
- Machine names, URLs, and image paths must be maintained in one data/constants source rather than duplicated in component markup. Localized copy belongs in Angular i18n messages.
