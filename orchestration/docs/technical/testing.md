# Testing

Use Vitest for unit and component tests and Playwright for browser-level E2E tests. Test key behavior and critical regressions; no numerical coverage percentage is required.

## Mutation testing

Use mutation testing to validate that unit and component tests detect meaningful changes in application behavior. Once the application and test tooling are scaffolded, run mutation tests on the relevant changed logic after the Vitest suite passes, prioritizing critical behavior such as language resolution, theme preferences, and form validation.

Review surviving mutants: add or improve tests for observable behavior that should have caught them, and document equivalent or otherwise non-actionable mutants. Report the results and unresolved survivors; do not treat a mutation score alone as proof of test quality or require a numerical threshold.

Tooling: evaluate Stryker with the Vitest runner in a time-boxed Stage 1 spike (`@stryker-mutator/core`, `@stryker-mutator/vitest-runner`, `stryker.config.json` with `testRunner: "vitest"`, `coverageAnalysis: "perTest"`, `mutate` limited to the language resolver, theme service and form validation; script `npm run mutation`). Use a plain Vitest config for mutation if the Angular unit-test builder is used for `npm test`. If it cannot run with zoneless component tests, restrict mutation to pure utilities and services and record the limitation. Ignore Stryker output folders in Git. Mutation runs as a pull-request CI gate after the unit tests (see [Technical requirements](./technical-requirements.md#pull-requests)) and at the Stage 5 gate. Keep it fast by limiting `mutate` to the critical files and using `--incremental` where supported.

## Translation validation

Provide `npm run i18n:check` (a dependency-free Node script) that:

1. Extracts messages with `ng extract-i18n` to a temporary file and compares ids with `src/locale/messages.pl.xlf`.
2. Fails on missing, stale or duplicate ids and on empty targets.
3. Fails when a target equals its source, except for an allowlist of brand and machine names.
4. Reports `[PLACEHOLDER]` targets as warnings, and as errors with `--strict` in the Stage 6 release gate.

Also set `i18nMissingTranslation` to `error` in `angular.json`. Prove the check with a negative test (removing one Polish unit must fail it) and unit-test the parser against fixture XLIFF files.

## Accessibility and performance automation

- Use `@axe-core/playwright` in a shared fixture with the `wcag2a`, `wcag2aa` and `wcag21aa` tags; any violation fails the test. Expose the suite as `npm run a11y`; it is a pull-request CI gate (see [Technical requirements](./technical-requirements.md#pull-requests)) and also runs locally at the Stage 3 and Stage 4 gates. Run it for `/pl` and `/en`, light and dark themes, mobile and desktop viewports, and with the mobile menu, a toast and the lightbox open. Exclusions need a written reason. Automated checks do not replace the manual keyboard, contrast and reduced-motion checks below.
- In Stage 6, run `@lhci/cli` against the production build for both locales: accessibility and SEO at least 0.95, best practices at least 0.9, CLS at most 0.1; record performance as a warning until a baseline exists. Lighthouse is not a required pull-request gate unless the CI section is updated.

## Traceability

| Requirement area | Unit/component | E2E | Other |
|---|---|---|---|
| Root language resolution | resolver util | `/` with seeded `localStorage` and browser locale | mutation |
| Direct route precedence | route test | `/pl` and `/en` with the opposite saved preference | |
| Language switch | switcher and hash | full navigation, hash, `lang`, storage | |
| Theme | service, toggle | reload persistence, no flash | axe both themes, mutation |
| Navigation and mobile menu | open, close, Escape, link | keyboard flow | axe |
| Machines, Pricing, Gallery, FAQ | content, links, alt text | section reachable and prerendered | `i18n:check` |
| Contact | validation and states | Formspree mocked for success and error | mutation |
| SSR and hydration | — | console and hydration listener | `ngh` markers in HTML |
| SEO | SEO service | metadata per locale | sitemap, robots, Lighthouse |
## Unit and component acceptance tests

Cover these behaviors:

- Polish and English route rendering with the correct page language.
- Root language resolution using saved preference, browser language, and Polish fallback.
- Root language resolution for `pl`, `pl-PL`, `en`, `en-US`, and unsupported browser locales.
- Direct `/pl` and `/en` rendering without redirecting to a saved or device-selected language.
- Language switching that performs a full-document navigation to the other localized output, preserves the current section hash, and updates `localStorage`; verify the destination document has the correct `<html lang>`.
- Theme defaults and persisted preferences, including `aria-pressed` and the theme attribute.
- Header navigation and mobile menu behavior, including link activation and Escape close.
- Pricing cards, machine cards, external machine links, and placeholder rendering.
- Gallery image alternatives and optional lightbox keyboard behavior.
- FAQ native accordion behavior.
- Contact form required fields, invalid values, loading state, success, and error notifications.
- Back-to-top visibility and keyboard activation.
- 404 route localization and return-home navigation.

## Querying template elements in component tests

- Add stable `data-testid` attributes to interactive elements that tests need to locate. Do not rely on styling classes or translated text as selectors.
- In each component test, group selectors in a local `util` object so test cases read as user actions and expectations.
- Use `fixture.debugElement.query(By.css(...))` when the test needs an Angular `DebugElement`; use `fixture.nativeElement.querySelector(...)` when it needs the actual DOM element or a nested rendered element.
- Include the test ID in the selector and return nullable elements where the DOM query can fail. Assert the element exists before interacting with it.

Example:

```ts
const util = {
  select(): DebugElement | null {
    return fixture.debugElement.query(By.css('[data-testid="model-select"]'));
  },
  selectTrigger(): HTMLElement | null {
    return fixture.nativeElement.querySelector(
      '[data-testid="model-select"] [data-pc-section="label"]'
    ) as HTMLElement | null;
  },
};
```

## Browser acceptance tests

The Playwright MCP server is available for interactive browser exploration when creating E2E tests. Its use is optional and does not replace automated Playwright test runs.

Verify:

- `/pl` and `/en` render the correct content and metadata.
- `/` redirects according to the saved preference or browser language.
- The English source locale and Polish XLIFF translation are both built and prerendered at `/en` and `/pl`, with the correct translated content and `<html lang>`.
- Switching language performs a full-document navigation to the other locale while preserving the current section hash and saved preference.
- Navigation smooth-scroll targets are reachable and section IDs exist.
- The mobile menu opens, closes, and remains keyboard operable.
- Theme and language preferences survive reload.
- The contact form displays validation and submission states without leaving an invalid submitted state behind.
- Key sections are present in prerendered HTML.
- The site has no hydration mismatch, console warning, or application stability error.
- Keyboard focus and visible focus states work for navigation, FAQ, modal/lightbox, and form controls.
- WCAG AA contrast and reduced-motion behavior are visually checked in dark and light themes.

## Hydration checks

- In dev mode, verify hydration stats in the browser console on `/pl` and `/en`.
- Confirm prerendered HTML contains hydration markers (for example `ngh` attributes).
- Check for no hydration mismatch and no “Application remains unstable” (NG0506) errors.
- Use Angular DevTools hydration overlay to verify any deferred interactive island stays dehydrated until its trigger, then hydrates without visual jumps.
- Playwright must fail on console errors or hydration warnings.

See [SSR and hydration](./technical-requirements.md#ssr-and-hydration) for implementation constraints.
