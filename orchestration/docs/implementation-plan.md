# Implementation plan

This roadmap divides implementation into validated stages. The [product requirements](product/product-requirements.md), [design direction](product/design-and-assets.md), [architecture](technical/architecture.md), [technical requirements](technical/technical-requirements.md), and [testing strategy](technical/testing.md) remain the sources of truth.

## Working agreement

- The Orchestrator defines a bounded task and its acceptance criteria, delegates implementation to Executor, and reviews the result and validation evidence.
- Complete each stage's gate before starting dependent work. Report the exact checks and results, unresolved placeholders, and blockers; do not mark a gate passed if a check was skipped or failed.
- Update the relevant status as work progresses. A stage may be split into smaller tasks, each with focused validation.
- Keep unconfirmed business details as marked placeholders. Do not copy reference-site branding or invent customer-facing facts.
- Commit small and often with `commit-changes`: after each logical implementation step that passes its checks (several commits per task are expected), so the history reads clearly step by step; do not batch steps into one commit. Pushes and pull requests require a separate user request.
- Follow the task breakdown, handoff template, and gate evidence rules below.

## Status

The application has not yet been scaffolded. All stages below are pending.

### 1. Foundation — pending

Scaffold the Angular application directly at the repository root (preserving `.github/` and leaving `orchestration/` untouched at the repository root), not in a subdirectory. Set up the documented runtime and npm versions, lockfile, SSR-capable build, Oxlint, Oxfmt, Vitest, Playwright, and local Husky hooks with Commitlint.

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

**Gate:** The documented build, lint, formatting, translation, accessibility (axe), unit, mutation (Stryker), and E2E gates pass; SSR, hydration, accessibility, and SEO acceptance checks are reviewed; remaining launch prerequisites and placeholders are reported.

## Stage task breakdown

Quality gate (QG) = `npm run build`, `npm run lint`, `npm run format:check`, `npm test` from the repository root. Add `npm run i18n:check` after Stage 2 and `npm run e2e` once the first E2E test exists; add `npm run a11y` once the first axe test exists (Stage 3). Tasks within a stage run in order; each is validated before the next.

### Stage 1 tasks

1. **Scaffold:** non-interactive `ng new` flags (standalone, zoneless, SSR); pinned Node/npm in `.nvmrc`, `engines`, and `packageManager`. Check: build; `npm ci` from a clean clone; application files live at the repository root, with `.github/` and `orchestration/` preserved (use `ng new` into a temporary directory and move files if needed).
2. **Tooling:** Oxlint, Oxfmt, Tailwind and scripts. Check lint and format; record what Oxlint/Oxfmt cover in Angular templates (`@if`, `@for`, `i18n`). Do not add ESLint or Prettier.
3. **Test runners:** Vitest smoke test, Playwright config and smoke test. Spike Stryker (see [Testing](technical/testing.md#mutation-testing)) and record the command or its limits.
4. **Hooks:** Husky pre-commit and commit-msg configured at the Git root (`.husky/` and `prepare` script in the root `package.json`). Prove on a temporary branch that a lint error is rejected, a non-conventional message is rejected, and a valid commit passes.
5. **CI skeleton:** `.github/workflows/` pull-request workflow running from the repository root (`npm ci`, `cache-dependency-path: package-lock.json`) with build, lint, format check and unit tests as sequential gates. Each later stage adds its gate (`i18n:check` in Stage 2, `a11y` in Stage 3, E2E with the first E2E test, mutation in Stage 5); Stage 6 finalizes the full documented pipeline and Lighthouse. No deployment step.

### Stage 2 tasks

1. **Localized builds:** English source at `/en`, Polish at `/pl`, both prerendered. Check `<html lang>` and `ngh` markers in the built output.
2. **Root resolution:** pure resolver plus redirect and `<noscript>` link. The localized build does not emit `/`, so a generated root page or host rewrite is required. Unit-test saved preference, `pl`, `pl-PL`, `en`, `en-US`, an unsupported locale, and a missing locale; SSR must not touch browser APIs.
3. **Localized 404:** lazy `**` route; one 404 per locale and a Polish fallback.
4. **i18n tooling:** `i18n:extract` and `i18n:check` (see [Testing](technical/testing.md#translation-validation)).
5. **Hydration baseline:** a shared Playwright fixture failing on console errors, hydration warnings and NG0506; prove it fails on a forced error.

### Stage 3 tasks

1. **Tokens and themes:** CSS variables from the business-card palette, deliberately designed light and dark, inline no-flash script touching only `<html>`. Check reload with a persisted theme and AA contrast.
2. **Theme service:** signals-based, SSR-safe, `aria-pressed`. Unit tests for system default, persisted preference, toggle.
3. **UI kit** in this order: Button, Card, SectionHeading, Icon, FormField, Accordion, Toast, Modal. One spec per control; Modal covers focus trap, Escape and focus return; Toast covers roles.
4. **Dependency rules:** a lint or script check that `shared/ui` imports nothing from pages or components; prove it fails on a violating import.

### Stage 4 tasks (each group has its own mini-gate)

- **4A Header and Hero:** navigation order, smooth scroll, mobile menu, language switcher, theme toggle, skip link, CSS-only record animation.
- **4B About and Machines:** machine data in one constants file; external link attributes; no link without a URL.
- **4C Gallery and Pricing:** responsive lazy images with translated alt text; 5 zł and 30 zł; two pricing cards only, no combo; lightbox per open question 2.
- **4D FAQ and Footer:** six placeholder pairs per language in native `<details>`; placeholder social links.

Per group: component specs, E2E reaching each section id, section content present in prerendered HTML, `i18n:check`, and axe on mobile and desktop in both themes and locales. Attach Playwright screenshots (390 px and 1440 px, both themes and locales) to the gate report as review evidence; there is no pixel-diff gate until a baseline exists.

### Stage 5 tasks

1. **Contact form:** unit tests for required, invalid email, loading, success, error and honeypot; E2E with Formspree mocked at the network layer, never a real request. Include a `[PLACEHOLDER]` privacy-notice line near the form (visitor data goes to Formspree), translated in both locales.
2. **Back to top:** unit and E2E keyboard activation.
3. **Preferences:** E2E that theme and language survive reload.
4. **Mutation run:** on the language resolver, theme service and form validation; record survivors.

### Stage 6 tasks

SEO service and metadata, sitemap, robots, favicon; README (Formspree setup, hosting prerequisites, any `ngSkipHydration` use); CI workflows running from the repository root with `cache-dependency-path: package-lock.json`; full regression, axe and Lighthouse reports; list of remaining placeholders and launch prerequisites (including the privacy policy). Security headers in `vercel.json` (CSP allowing the inline theme script by hash, `X-Content-Type-Options`, `Referrer-Policy`); verify the no-flash theme script still works under the CSP.

## Decisions

Defaults chosen while the user was unavailable; change them here before the affected stage.

- **Gate review:** Orchestrator plus an independent read-only review of the stage diff before a gate is declared passed.
- **Placeholder marker:** non-final copy starts with `[PLACEHOLDER]` in each language; non-copy placeholders (URLs, email, social links) use constants named with a `PLACEHOLDER_` prefix and a comment.
- **Accessibility and performance automation, mutation tool, Vercel previews:** see [Testing](technical/testing.md) and [Technical requirements](technical/technical-requirements.md#preview-deployments).
- **Production hosting:** undecided; no auto-deploy workflow without separate approval. Stage 2 stays host-agnostic: a generated root page plus `vercel.json` for previews, with the rewrite rules any other static host would need documented in the README.
- **Definition of done:** a task is done only when each item is evidenced in the gate report (a skipped check means not done): acceptance criteria marked met/unmet; the quality gate passed; new behavior has a meaningful test and bug fixes a regression test; SSR safety (guarded browser APIs, `ngh` markers); i18n markers with Polish translated or `[PLACEHOLDER]`; accessibility (roles, keyboard, focus, reduced motion) checked for touched UI; dependency direction respected; no invented business facts; Orchestrator re-ran the checks and inspected the diff.
- **Defaults applied while the user was unavailable:** lightbox is deferred from the first release (Modal is still built); logo is recreated as SVG from the business card; the independent stage reviewer is the read-only `code-review` agent.

## Open questions

Ask once, consolidated, in the planning phase of the stage that needs the answer.

1. **Stage 2:** production hosting for the root page and per-locale 404 (previews use Vercel).
2. **Stage 4:** is the lightbox in scope for the first release?
3. **Stage 6:** are the domain, DNS and contact email confirmed?
4. **Stage 3:** is a vector logo available, or should the mark be recreated from the business card?

## Orchestration rules

- Exploration: no user questions; record unknowns. Planning: consolidated essential questions only.
- One Executor handoff per task. The Orchestrator inspects the diff and re-runs the task checks; at most two correction cycles per task, counted across Executor and Orchestrator.
- After a stage's tasks, run the full gate, collect evidence, and get the independent review. Fix findings within the cycle limit.
- Orchestrator may use `branch-naming` at the start of a stage and `commit-changes` after a task passes its checks, reporting each action to the user. `push-changes` (requires a successful `npm run build`) and `pull-request-description` still need an explicit user request.
- The agent definitions pin specific models; if one is unavailable, report it instead of substituting silently.

### Executor handoff template

```text
Outcome and reason:
Acceptance criteria (observable):
In scope (paths) / Out of scope:
Source docs and findings (exact paths):
Constraints: standalone, OnPush, zoneless, SSR/hydration safe, a11y, i18n markers, Vitest, Oxlint/Oxfmt, no ESLint/Prettier
Checks to run (exact commands) and known baseline:
Confirmed decisions / Unresolved questions (do not guess):
Correction cycle: n of 2
Report format: status; changed files; criteria met/unmet; commands and results; blockers; follow-up
```

### Gate evidence report

```text
Stage / task:
Revision and Node/npm versions:
Commands run -> result, in order:
Skipped checks and why (a skip means the gate is not passed):
Manual checks (viewport, theme, locale, keyboard) and findings:
Independent review findings and resolution:
Placeholders still open:
Correction cycles used:
```

## Risks to watch

- The localized build emits only `/en` and `/pl`: `/` and the per-locale 404 need explicit handling.
- Scaffolding at the repository root may overwrite or conflict with existing files (`README.md`, `.gitignore`); merge them deliberately.
- Oxlint and Oxfmt may not fully cover Angular templates.
- The "fail on console errors" E2E rule conflicts with the placeholder Formspree URL unless the request is mocked.
- Hydration console checks need the dev or SSR build; production builds do not log them.
- Stryker's Vitest runner needs a plain Vitest config and may not run zoneless component tests; fallback is a separate plain-Vitest config for pure logic only, with the limitation recorded.
- Oxfmt/Oxlint may not cover Angular templates; record the actual coverage in Stage 1 and rely on strict template compilation and `i18n:check` for the rest.
- Merge-to-main skips lint, translations, axe and mutation by design; the preceding pull request already gates them.
- `LocalBusiness` JSON-LD needs address, phone and email; keep them as `PLACEHOLDER_` constants and report them as launch prerequisites.

## Skills

`branch-naming` when starting a stage and `commit-changes` after a task passes its checks may be used by Orchestrator without a separate request; `push-changes` only on explicit request and after a successful `npm run build`; `pull-request-description` only on explicit request, at the end of a stage using the gate evidence report. The `playwright-explore-website` and `playwright-generate-test` skills are optional aids for E2E authoring; automated Playwright runs remain the evidence.
