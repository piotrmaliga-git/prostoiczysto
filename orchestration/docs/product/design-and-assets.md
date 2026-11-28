# Design direction and assets

This document contains the visual direction and source-asset guidance. Product behavior is specified in [Product requirements](./product-requirements.md).

## Design references

Use the business card image at `../images/prosto_i_czysto_wizytówka.png` as the source of truth for the brand identity:

- Take the website's brand colors from the card and map them to CSS variables for dark and light themes. Do not take colors from inspiration websites.
- Reuse or recreate the logo/brand mark as SVG if possible.

Use the websites and concepts in [Secondary inspiration](#secondary-inspiration) as references for what makes an effective website: layout and section composition, shapes, spacing, interaction patterns, and user experience. Adapt useful ideas to this project rather than copying a reference site. Their color palettes and branding are not references for this project.

## Image assets

- Use `../images/photo_ex.jpg` (workspace path: `orchestration/docs/images/photo_ex.jpg`) as the shared temporary photo placeholder for About, Machines, and Gallery until proper photos are available.
- Copy it into the app's public image assets as needed; use responsive sizing and cropping.
- Do not substitute unrelated stock photos or create replacement photo illustrations. SVG is appropriate for icons and the logo.
- Source assets are in the workspace `orchestration/docs/images` folder. Do not use the available machine-specific photos for page sections; use only the shared `photo_ex.jpg` placeholder until final photos are provided.

## Visual direction

Apply the business-card palette consistently across both themes, maintaining accessible contrast. Do not assume a dark, moody palette or prescribe colors independently of the card.

- Vinyl motifs: record grooves, a slowly rotating record in the hero (CSS animation), and subtle label-like circular elements.
- Consider bold editorial typography (strong display font plus clean sans body), generous spacing, and large imagery where it fits the brand and page structure.
- Subtle, smooth scroll-triggered reveals, restrained hover and menu transitions.
- Keep motion polished but not distracting; avoid effects that harm performance or cause layout shifts.
- Reduce or disable nonessential motion under `prefers-reduced-motion`.
- Design both themes deliberately; the light theme should not be a simple inversion of the dark theme.

## Secondary inspiration

Links may not open; use the notes above if they do not:

- <https://dribbble.com/shots/27151334-Records-Vinyl-Store-Landing-Page>
- <https://dribbble.com/shots/23913880-Spinlab-Music-Vinyl-Company-Landing-Page-Website>
- <https://dribbble.com/shots/26342459-Vinyl-Listening-Landing-Page-Concept>
- <https://shelter.pl/> — good animations.
- <https://lecimyposwoje.pl> — good animations and page structure.

### Screen inspirations

Screenshots inspired by www.lecimy.pl are stored in `../images/inspirations` (workspace path: `orchestration/docs/images/inspirations`). Use them as visual references for layout, composition, and interaction ideas only, not for colors or branding.
