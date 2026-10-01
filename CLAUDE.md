# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Shared PDF deck control for live webinar production ("deck-sync"). Two self-contained static HTML files, no backend, no build step, no dependencies to install — state syncs through a Supabase Realtime **broadcast channel** (no tables, no migrations).

- `viewer.html` — fullscreen PDF render, used as an OBS browser source (1920×1080)
- `controller.html` — presenter remote (phone or browser tab); any number can be open at once

Hosted as static files (GitHub Pages — `.nojekyll` is present and Jekyll is deliberately disabled).

## Development

There is no build, lint, or test tooling. Edit the HTML files directly and open them in a browser to test:

```
open "viewer.html?room=test&pdf=https://…/deck.pdf"
open "controller.html?room=test&pdf=https://…/deck.pdf&name=Andrii"
```

Testing sync behavior requires two browser windows (one viewer, one controller) pointed at the same `?room=`.

## Architecture

Both files load pdf.js and supabase-js from CDNs and share the same pattern:

1. Read `room`, `pdf` (and for the controller, `name`) from the query string.
2. Subscribe to Supabase channel `deck:<room>` with `broadcast: { self: false }`.
3. Render PDF pages to `<canvas>` via pdf.js, scaled by `devicePixelRatio`.

Sync protocol (three broadcast events):

- `goto` `{ page, who }` — sent by a controller on any navigation; both viewer and other controllers re-render to that page. Last click wins.
- `sync-request` `{}` — sent by any client on subscribe; any live controller answers with a `goto` carrying its current page so late joiners catch up.
- `ink` `{ page, id, color, pts: [[x,y],…] }` — drawing, sent by a controller in pen mode (✎ DRAW button or `D` key). Points are normalized 0..1 to the page, batched every 60ms, and appended to stroke `id`. Receivers ignore ink for pages they aren't showing. Strokes fade out on their own (visible 2.5s after their last point, then a 1s fade), are cleared locally on page change, and are never replayed to late joiners. Color is derived from the presenter's `name`.

The controller additionally tracks "who is driving" (`driver` label + red tally dot when it's you, auto-clearing after 5s) and renders a clickable thumbnail strip.

Constraints to preserve when editing:

- `SUPABASE_URL` / `SUPABASE_ANON_KEY` are hardcoded near the top of **both** files under the "EDIT THESE TWO LINES" banner — keep them in sync.
- Each file must stay a single self-contained HTML file (inline CSS/JS, CDN scripts only).
- The PDF URL must be CORS-accessible (a public Supabase Storage bucket works).
- Page renders cancel any in-flight `renderTask` before starting a new one — keep this to avoid pdf.js concurrent-render errors.
- No auth beyond the room name by design; rooms are unguessable strings shared with trusted presenters.
