# Site Builder

A brief becomes a working one page site in minutes, in the client's own colors and language.

Live: [genvidpro.com/builder](https://genvidpro.com/builder)

## How it works

1. The client answers a short brief: business, tone, colors, language, call to action.
2. `tpl-engine.js` picks a layout family and fills every block from that brief, including copy length rules so nothing overflows on a phone.
3. The result is a single self contained HTML file: inline styles, no build step, no dependency to install. It can be hosted anywhere or handed over as a file.

## Design decisions

- **One file out.** A small business owner should be able to move their site without a developer. No node_modules, no framework runtime.
- **RTL first, not RTL later.** Hebrew and Arabic switch the whole grid, not just text direction. Carousels, arrows and form validation mirror with it.
- **Copy that fits.** The engine measures rendered length per block and trims or expands the generated copy rather than letting text break the layout.
- **Accessible defaults.** Contrast is computed from the chosen brand color; if a pair fails, the engine shifts the shade instead of shipping unreadable text.

## Files

- `builder.html` — the builder interface: brief form, live preview, export
- `tpl-engine.js` — 88 KB template engine: layout families, color logic, copy fitting, export

## Status

In production use for GenVidPro clients. Part of [genvidpro-site](https://github.com/romachorny/genvidpro-site), published separately because it stands on its own.
