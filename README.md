# Prosto i Czysto — project overview

This is the entry point and source map for the Prosto i Czysto landing-page project. The deliverable is a complete, runnable app in `prostoiczysto`, with a README.

## Start here

The site promotes vinyl record cleaning and flattening. Polish is the primary language and English is the second. Use marked placeholder copy for all marketing text; do not invent final customer-facing wording.

### Product and business documentation

- [Product requirements](docs/product/product-requirements.md) — Business facts, routes, page sections, shared UI kit, and visitor-facing features.
- [Design and assets](docs/product/design-and-assets.md) — Visual direction and source-image guidance.
- [Open questions](docs/product/open-questions.md) — Unconfirmed business details that must remain placeholders.

### Technical documentation

- [Technical architecture](docs/technical/technical-architecture.md) — Stack, architecture, hydration, SEO, accessibility, and tooling.

### Development agents

- [Orchestrator](.github/agents/orchestrator.agent.md) — Coordinates implementation requests and passes a self-contained task context to Executor.
- [Executor](.github/agents/executor.agent.md) — Implements the approved scope and returns changes and validation evidence.

## Operating rules

- Use the technical architecture as the implementation authority for the Angular stack, SSR, hydration, SEO, accessibility, and tooling.
- Make implementation decisions in small, independently verifiable steps.
- Run relevant build, lint, unit, and browser checks after changes.
- If something is ambiguous or an asset is missing or unreadable, do not guess silently: before the app is scaffolded, record the assumption in workspace `docs/assumptions.md`; after Phase 1, use `prostoiczysto/docs/assumptions.md`.
- Keep the implementation simple, follow Angular conventions, separate responsibilities appropriately, and prefer maintainable solutions over unnecessary complexity.

## Precedence

- The architecture diagram at `docs/images/architektura.png` wins for folder structure and layering.
- The written product and technical requirements win for behavior.
- If a relevant detail is not confirmed, preserve it as a placeholder and track it in [Open questions](docs/product/open-questions.md).
- Keep each requirement in its designated document and link to it instead of duplicating it elsewhere.

## Source assets

Source assets are in `docs/images/`:

- `docs/images/architektura.png` — architecture diagram.
- `docs/images/prosto_i_czysto_wizytówka.png` — primary visual reference.
- `docs/images/photo_ex.jpg` — shared placeholder image for the About, Machines, and Gallery page sections until final photos are available.

See [Design and assets](docs/product/design-and-assets.md) for how to use them in the app.
