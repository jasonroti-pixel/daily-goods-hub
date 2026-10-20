# Daily Goods Hub (prototype preview)

Interactive look-and-feel prototype for THE DAILY GOODS show hub.
This preview is a single self-contained HTML file (`index.html`).

The production build (per `SPEC.md`) will replace this with the real
multi-module app. Do not treat this file as the app; it is the design
and interaction reference.

Public preview ships with the Gemini API key redacted; the key is
pasted at launch.

## Shared state (Supabase, "Route B")

The Builder, Newspaper, Corkboard, Archive and feed "story of interest" flags are shared with the
whole show team in real time. The Keep, Tools, settings and the Gemini key stay on each device.

- **Local first.** localStorage is still the instant store; the UI never waits on the network. The
  sync layer (bottom of `index.html`, search `Shared real-time state`) wraps the existing save
  functions and mirrors each surface to one row of `public.hub_docs`:
  `builder:<YYYY-MM-DD>`, `newspaper:<YYYY-MM-DD>`, `corkboard`, `archive`, `feed-flags`.
- **Uploads:** about 800 ms after a local edit. **Downloads:** on load, on reconnect, and live via
  Realtime (`postgres_changes`). Our own echoes are ignored by payload hash.
- **Conflicts (v1):** last write wins per document. If you are typing when a teammate's change to
  the same document arrives, it waits until you leave the field, is applied, and your edited fields
  are re-applied on top, so both edits survive. Two uploads of the same document within about a
  second of each other still resolve to one of them.
- **Offline:** if Supabase or the CDN can't be reached the hub works exactly as before (local only),
  retries every 30 s and on reconnect, and uploads pending edits when it can. Header dot: green =
  synced, amber = local only.
- **Keys:** only the public (publishable) key is used. There are no accounts, so anyone with the
  page can read and write these documents; remote HTML and links are sanitized before rendering.
  Never put a service_role / secret key in this repo.
- The first time a device syncs a document it never synced before, and the shared copy replaces
  local content, the local copy is kept once under `dg_sync_backup:<doc>` (three at most).
