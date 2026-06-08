# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Single-page static HTML crowdfunding site for Morning Storm, a Brazilian stoner metal band funding a European tour in January 2027. No build step, no framework, no dependencies — one file, inline CSS, no JavaScript beyond what the browser provides.

## Development

Open `home.html` directly in a browser. No server required unless you need to test CORS-sensitive features (the hero video loads from an external CDN).

## Architecture

Everything lives in `home.html`: all CSS is in a `<style>` block, all markup follows after. The page uses three Google Fonts (Anton for display, DM Sans for body, JetBrains Mono for labels/tags) and a CSS custom property palette defined on `:root`.

**Design tokens (`:root`):**
- `--ink` / `--ink-2` / `--concrete` — dark background layers
- `--line` — border color
- `--smoke` — muted text
- `--bone` — primary text
- `--glow` / `--glow-2` — orange accent (calls to action, highlights)
- `--rust` — secondary accent used in the fundraising bar gradient

**Utility classes:** `.mono` (JetBrains Mono label style), `.display` (Anton display style), `.wrap` (max-width container), `.btn` / `.btn-ghost`, `.rv` + `.d1–.d4` (reveal animations).

**Sections in order:** nav → hero (with video bg + fundraising bar) → #pitch → #music → #tiers → #lineup → #route → #join (email capture) → footer.

## Content placeholders

Several fields in the current markup are intentionally left as placeholders and need real content before launch:
- Band member names and photos (`[Name]`, `[ photo ]`)
- Lineup blurb at the bottom of `#lineup`
- Streaming/social links (`href="#"`)
- Hero video source (currently a Midjourney CDN URL, not guaranteed permanent)
- Tour date venues marked `TBC`
- Budget line items with `[N]` tokens in `#pitch`
- EP year `[year]` in the hero tag
