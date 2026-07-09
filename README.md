# deck-sync

Shared PDF deck control for live production. Two static files, no backend —
state syncs through a Supabase Realtime broadcast channel.

- `viewer.html` — fullscreen render, add as OBS browser source (1920×1080)
- `controller.html` — presenter remote (phone or browser tab), any number of them

## Setup (once)

1. In both files, set `SUPABASE_URL` and `SUPABASE_ANON_KEY` (Project Settings → API).
   Broadcast channels need no tables and no migrations.
2. Host the two files anywhere static: Supabase Storage public bucket, Vercel,
   or an LXC on the Proxmox box behind Tailscale Funnel if presenters are external.
3. Upload the deck PDF somewhere CORS-friendly. A public Supabase Storage
   bucket works out of the box.

## Per webinar

- OBS browser source:
  `viewer.html?room=webinar1&pdf=https://…/deck.pdf`
- Each presenter gets:
  `controller.html?room=webinar1&pdf=https://…/deck.pdf&name=Andrii`

Arrow keys / space / PageUp-PageDown work on the controller, so a hardware
clicker pointed at the controller tab also works. Last click wins; the header
shows who is driving, and the tally dot lights red when it's you.

## Known limits

- No auth beyond the room name. Anyone with the link controls the deck —
  fine for trusted presenters, use an unguessable room string.
- Supabase free tier: 200 concurrent Realtime connections. A webinar uses
  one per viewer + one per controller. Not a constraint here.
- PDF only. For a live Google Sheets segment, run the sheet fullscreen on
  the compositor and switch scenes — syncing an embedded sheet's scroll
  state is not worth the jank.
