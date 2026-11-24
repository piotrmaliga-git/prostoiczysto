# Prosto i Czysto — project overview

This is the repository-level entry point and source map for the Prosto i Czysto landing-page project. The Angular application belongs in the repository-root `prostoiczysto/` directory; keep project documentation and GitHub configuration at the repository root.

## Start here

The site promotes vinyl record cleaning and flattening. Polish is the primary language and English is the second. Use marked placeholder copy for all marketing text; do not invent final customer-facing wording.

### Product and business documentation

- [Product requirements](docs/product/product-requirements.md) — Business facts, routes, page sections, shared UI kit, and visitor-facing features.
- [Design and assets](docs/product/design-and-assets.md) — Visual direction and source-image guidance.
### Technical documentation

- [Project architecture](docs/technical/architecture.md) — Repository structure, Angular layers, responsibilities, and dependency direction.
- [Technical requirements](docs/technical/technical-requirements.md) — Stack, SSR/hydration, CI/CD, SEO, and accessibility.
- [Testing](docs/technical/testing.md) — Key unit, component, browser, and hydration acceptance checks.

### Development agents

- [Orchestrator](.github/agents/orchestrator.agent.md) — Coordinates implementation requests and passes a self-contained task context to Executor.
- [Executor](.github/agents/executor.agent.md) — Implements the approved scope and returns changes and validation evidence.

## Operating rules

- Use the technical architecture as the implementation authority for the Angular stack, SSR, hydration, SEO, accessibility, and tooling.
- Pull requests run build, Oxlint/Oxfmt lint and format checks, translation validation, unit, and E2E checks as sequential gates. Merges to `main` run build, unit, and E2E checks sequentially. Any failed gate stops the pipeline.
- Make implementation decisions in small, independently verifiable steps.
- Run relevant build, lint, unit, and browser checks after changes.
- If something is ambiguous or an asset is missing or unreadable, do not guess silently: preserve the relevant placeholder and record it only in the designated product or technical document.
- Keep the implementation simple, follow Angular conventions, separate responsibilities appropriately, and prefer maintainable solutions over unnecessary complexity.
- Keep source code and developer-facing identifiers in English. Add customer-facing copy to `prostoiczysto/src/assets/i18n/en.json` first and translate it into `prostoiczysto/src/assets/i18n/pl.json` under the same keys.
- Direct `/pl` and `/en` routes determine the displayed language. On `/`, match Polish or English device browser languages and fall back to Polish for unsupported locales.

## Precedence

- The textual project structure in [Project architecture](docs/technical/architecture.md) wins for folder structure and layering.
- The written product and technical requirements win for behavior.
- If a relevant detail is not confirmed, preserve it as a placeholder in the relevant product or technical document.
- Keep each requirement in its designated document and link to it instead of duplicating it elsewhere.

## Source assets

Source assets are in `docs/images/`:

- `docs/images/prosto_i_czysto_wizytówka.png` — primary visual reference.
- `docs/images/photo_ex.jpg` — shared placeholder image for the About, Machines, and Gallery page sections until final photos are available.

See [Design and assets](docs/product/design-and-assets.md) for how to use them in the app.
