# Testing

Use Vitest for unit and component tests and Playwright for browser-level E2E tests. Test key behavior and critical regressions; no numerical coverage percentage is required.

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
