# Implementation plan

This roadmap divides implementation into validated stages. The [product requirements](product/product-requirements.md), [design direction](product/design-and-assets.md), [architecture](technical/architecture.md), [technical requirements](technical/technical-requirements.md), and [testing strategy](technical/testing.md) remain the sources of truth.

## Working agreement

- The Orchestrator defines a bounded task and its acceptance criteria, delegates implementation to Executor, and reviews the result and validation evidence.
- Complete each stage's gate before starting dependent work. Report the exact checks and results, unresolved placeholders, and blockers; do not mark a gate passed if a check was skipped or failed.
- Update the relevant status as work progresses. A stage may be split into smaller tasks, each with focused validation.
- Keep unconfirmed business details as marked placeholders. Do not copy reference-site branding or invent customer-facing facts.
- Commits and pushes require a separate user request.

## Status

The application has not yet been scaffolded. All stages below are pending.

### 1. Foundation — pending

Scaffold the Angular application in the repository-root `prostoiczysto/` directory. Set up the documented runtime and npm versions, lockfile, SSR-capable build, Oxlint, Oxfmt, Vitest, Playwright, and local Husky hooks with Commitlint.

**Gate:** The application builds; lint and format checks pass; the test runners can execute a focused smoke test; the local pre-commit and commit-msg hooks enforce the documented checks. Record the commands and results.

### 2. Rendering and languages — pending

Implement localized `/pl` and `/en` outputs, root language selection, the localized 404, SSR, and hydration according to the product and technical requirements.

**Gate:** Both localized outputs build and prerender with the correct language; direct routes and root selection behave as specified; focused tests pass and browser checks show no hydration errors.

### 3. Design foundation — pending

Establish responsive layout, the brand-based light and dark themes, and the shared UI kit before page sections depend on it. Preserve accessible and hydration-safe behavior.

**Gate:** Relevant component tests pass; keyboard behavior, contrast, reduced motion, and responsive presentation are checked in both themes.

### 4. Landing-page sections — pending

Build the documented page sections in small groups, using the specified placeholder content and assets. Validate each group before adding dependent work.

**Gate:** The required sections and navigation appear in the correct order on both localized routes; focused tests and responsive browser checks pass for each group.

### 5. Interactive behavior — pending

Complete and verify navigation, preferences, contact submission, and other required interactions, coordinating with earlier stages where these behaviors are introduced.

**Gate:** Unit and E2E tests cover successful and error states, keyboard behavior, and persisted preferences. Run focused mutation testing for relevant critical logic and review surviving mutants as described in the testing strategy.

### 6. Release readiness — pending

Complete SEO, accessibility, translation validation, CI workflows, documentation, and remaining regression checks. Keep deployment and unconfirmed business details within the documented boundaries.

**Gate:** The documented build, lint, formatting, translation, unit, and E2E gates pass; SSR, hydration, accessibility, and SEO acceptance checks are reviewed; remaining launch prerequisites and placeholders are reported.
