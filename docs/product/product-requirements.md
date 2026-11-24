# Product requirements

This document defines what the Prosto i Czysto landing page must contain and how its visitor-facing features behave. Technical implementation constraints live in [Technical requirements](../technical/technical-requirements.md).

## Business and audience

- Business: vinyl record cleaning and flattening warped records.
- Brand: **Prosto i Czysto**.
- Primary audience and language: Polish; English is the second language.
- During implementation, developer-facing source text and technical identifiers must remain in English.
- Author and maintain `en.json` first. Polish copy must be added in `pl.json` as an explicit translation of the same English keys.
- Do not ship untranslated English customer-facing copy on `/pl`, or untranslated Polish customer-facing copy on `/en`.
- Use clearly marked Lorem ipsum-style placeholders in Polish and English for all customer-facing marketing copy, including hero and section text, descriptions, and FAQ questions and answers. Do not invent final marketing wording.
- Keep supplied facts accurate, including business and machine names, prices, and form requirements. Functional UI labels and accessibility text must be clear and translated.

## Routes and navigation

- `/pl` and `/en`: the full landing page, prerendered as static HTML for each language.
- `/`: client-side language selection in this order:
  1. Language saved in `localStorage`.
  2. Browser/device language (`navigator.language`) if it indicates Polish or English.
  3. Polish otherwise.
- Polish device languages such as `pl` or `pl-PL` open `/pl`.
- English device languages such as `en` or `en-US` open `/en`.
- All other browser/device languages open `/pl` by default.
- Direct `/pl` and `/en` URLs always take precedence over the saved preference and device language.
- The prerendered root page cannot detect browser language. Include a lightweight client redirect, a `<noscript>` link, and a `<meta http-equiv="refresh">` fallback to `/pl`.
- `**`: lazy-loaded, styled 404 page with a link back home. Localize it when the URL starts with `/pl` or `/en`; otherwise use Polish.
- Language switching navigates between `/pl` and `/en`, preserves the current hash/section, and saves the choice.
- Header navigation order: About, Machines, Gallery, Pricing, FAQ, Contact.

## Page sections

Keep the section order below. It follows the intended landing-page flow: hook, introduction, equipment, proof, price, answers, call to action, footer.

1. **Sticky header and hero**
   - Navigation links smooth-scroll to sections.
   - Accessible mobile hamburger menu; close it on link click and Escape.
   - Language switcher and theme toggle.
   - Primary CTA scrolls to Contact; secondary CTA scrolls to Pricing.
2. **About**
   - Use `docs/images/photo_ex.jpg` as the temporary photo placeholder.
   - Include clearly marked Polish and English placeholder copy.
3. **Machines**
   - One card per machine, using `docs/images/photo_ex.jpg` as the temporary photo placeholder, with name, translated placeholder description, and product link where available.
   - Cleaning:
     - HumminGuru NOVA Advanced Ultrasonic Vinyl Record Cleaner: <https://humminguru.com/products/humminguru-nova>
     - HumminGuru Water Dispenser (simplifies refilling water from the tank to the basin); no product URL supplied, so use a clearly marked placeholder constant.
   - Flattening:
     - ORB DF-01iA+ high-end vinyl record flattener; no product URL supplied, so use a clearly marked placeholder constant.
   - Keep machine names, URLs, and image paths in one constants file.
   - External links use `target="_blank"` and `rel="noopener noreferrer"`. Render no link when a card has no URL.
   - Add Machines to navigation, translations, and JSON-LD/sitemap where relevant.
4. **Gallery**
   - Use responsive images sourced from `docs/images/photo_ex.jpg` as temporary photo placeholders; do not create substitute photo illustrations.
   - Use responsive images, lazy loading, and meaningful translated alt text.
   - An optional lightbox may use the shared Modal and must support Escape, arrow-key navigation, focus trapping, and focus return to the trigger.
   - Gallery intentionally precedes Pricing: show results first, then price.
5. **Pricing**
   - Three cards: cleaning, flattening, and combo. Prices are per record, in PLN:
     - Cleaning: **5 zł**
     - Flattening: **30 zł**
     - No combo price
   - Each card has a CTA that scrolls to Contact.
6. **FAQ**
   - Accessible accordion with six clearly marked placeholder question/answer pairs in both languages. Cover service time, eligible records, suitable warps, handover or shipping, larger-batch discounts, and risks or guarantees.
   - Use the shared Accordion built on native `<details>`/`<summary>`. Only one open item at a time is optional.
   - Keep FAQ translation keys in one constants file; adding a question means adding one key pair per language.
   - Add FAQ to navigation and translations; an `FAQPage` JSON-LD block is optional.
7. **Contact**
   - Form fields: name, email, message.
   - Validate the form and show success/error notifications.
8. **Footer**
   - Social media links are placeholders.

## Shared UI kit

There is no existing design-system library. Build a small internal kit in `src/app/shared/ui` before sections use it. Components must be standalone, OnPush, accessible by default, and support `data-testid`.

- Button: primary, secondary, and ghost variants; sizes; loading/disabled states; optional `<a>` rendering.
- Modal: focus trap, Escape to close, focus returns to trigger; reused by the gallery lightbox.
- Toast/Notification: service-driven, `role="status"` / `role="alert"`, auto-dismiss.
- Card.
- SectionHeading.
- FormField: label, control, error, and `aria-describedby`.
- Icon.
- Accordion: native `<details>` / `<summary>`, used by FAQ.

All kit components must follow the hydration-safe rules in [Technical requirements](../technical/technical-requirements.md).

## Visitor-facing features

### Internationalization

- Polish and English runtime switcher.
- Language is determined by the URL (`/pl`, `/en`); `/` chooses automatically as specified above.
- Save the selected language in `localStorage`.
- Update `<html lang>` on language switch and navigation.
- Provide placeholder copy in both languages.

### Theme

- Dark and light themes; default to system preference.
- Manual toggle, with preference saved in `localStorage`.
- Prevent a flash of the wrong theme with a small inline script in `index.html` that sets the theme attribute on `<html>` before first paint. Keep it SSR- and hydration-safe.
- Meet WCAG AA contrast in both themes.

### Contact form

- Submit to Formspree.
- Endpoint URL comes from environment config and initially uses `https://formspree.io/f/YOUR_FORM_ID`.
- Include a hidden honeypot field, a disabled/loading state while submitting, and clear error messages.
- README must explain that the owner needs to create the Formspree form and connect it to `prostoczystowinyle@gmail.com` in the dashboard. This email is currently unconfirmed and must remain an explicitly marked placeholder until it is verified.

### Back to top and responsive behavior

- Fixed bottom-right back-to-top button appears after scrolling past the end of About; it smooth-scrolls and is keyboard accessible.
- Design mobile-first and work across screen sizes.
