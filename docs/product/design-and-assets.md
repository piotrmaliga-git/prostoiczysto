# Design direction and assets

This document contains the visual direction and source-asset guidance. Product behavior is specified in [Product requirements](./product-requirements.md).

## Primary reference

Use the business card image at `../images/prosto_i_czysto_wizytówka.png` as the primary design reference:

- Color palette, with hex values mapped to CSS variables for dark and light themes.
- Fonts or closest free Google Fonts equivalents.
- Logo/brand mark treatment, layout patterns, shapes, spacing, and tone.

Reuse its visual language across the hero, section headings, cards, and footer. Reuse or recreate the logo as SVG if possible.

## Image assets

- Use `../images/photo_ex.jpg` (workspace path: `docs/images/photo_ex.jpg`) as the shared temporary photo placeholder for About, Machines, and Gallery until proper photos are available.
- Copy it into the app's public image assets as needed; use responsive sizing and cropping.
- Do not substitute unrelated stock photos or create replacement photo illustrations. SVG is appropriate for icons and the logo.
- Source assets are in the workspace `docs/images` folder. Do not use the available machine-specific photos for page sections; use only the shared `photo_ex.jpg` placeholder until final photos are provided.

## Visual direction

If the business card conflicts with the following notes, the business card wins.

- Dark, moody base with warm cream text and one accent color.
- Vinyl motifs: record grooves, a slowly rotating record in the hero (CSS animation), and subtle label-like circular elements.
- Bold editorial typography (strong display font plus clean sans body), generous spacing, and large imagery.
- Subtle, smooth scroll-triggered reveals, restrained hover and menu transitions.
- Keep motion polished but not distracting; avoid effects that harm performance or cause layout shifts.
- Reduce or disable nonessential motion under `prefers-reduced-motion`.
- Light theme should feel like warm paper/cream, not simply an inversion of dark.

## Secondary inspiration

Links may not open; use the notes above if they do not:

- <https://dribbble.com/shots/27151334-Records-Vinyl-Store-Landing-Page>
- <https://dribbble.com/shots/23913880-Spinlab-Music-Vinyl-Company-Landing-Page-Website>
- <https://dribbble.com/shots/26342459-Vinyl-Listening-Landing-Page-Concept>
- <https://shelter.pl/> — good animations.
- <https://lecimyposwoje.pl> — good animations and page structure.
