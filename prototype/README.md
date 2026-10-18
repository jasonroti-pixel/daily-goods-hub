# Prototype reference files (key redacted)

- `prototype-v15-header-swap.html`: the LOCKED visual direction ("Royal Signal").
  Match this look exactly: four blues, saturated blue header, white cards,
  deep indigo lanes, Fraunces/Geist/Space Mono.
- `prototype-v19-feed-history.html`: functional reference for the Feeds tab.
  Real scout JSONs, 5-day rolling history, three card actions, Saturday
  auto-wipe, no source labels.
- `prototype-v20-builder-layout.html`: functional reference. Builder
  Standard (default) vs Grid layout toggle, per-card Copy for Sheets.
- `prototype-v21-rundown-format.html`: functional reference. Builder
  Standard mode mirrors the sheet anatomy (TOPIC rows, TEASE rows, HOUR 2,
  fixed header/footer furniture), in-app print preview.
- `prototype-v22-tools.html`: functional reference. Tools & Resources tab
  (JayStation, Story Scout Next, Nico Fetcher cards + add/edit/delete),
  Settings page with browser-only API key management (no key in repo).
- `prototype-v23-artcraft.html`: functional reference. Adds ArtCraft as a
  fourth default tool card with its paid-software replacement list; defaults
  merge with tombstones so deleted cards stay deleted.
- `prototype-v24-osint.html`: functional reference. Adds OSINT4ALL
  (start.me directory of free OSINT tools) as a fifth default tool card.
- `prototype-v25-review1.html`: functional reference. Zoom link/password
  locked read-only in the builder (editable in Settings); Today on the Show is
  a manual textarea with reset-to-auto.
- `prototype-v26-framextv.html`: functional reference. Adds frameXTV
  (free movie/TV/anime streaming, clip and audio hunting) as a sixth default
  tool card.
- `prototype-v27-review2.html`: functional reference. Per-lane
  Interview/Talk type switch (both payloads preserved), delete on every card,
  undo/redo with header buttons and keyboard shortcuts.
- `prototype-v28-feed-refresh.html`: functional reference. Weekly feed
  refresh moved to Sunday 12:00 PM Toronto (was Saturday midnight); Saturday is
  now a normal fetch day.
- `prototype-v29-linkfix.html`: functional reference. URLs pasted without
  a scheme are normalized to https:// at save and render time (builder, feeds,
  newspaper, tools, Zoom link); newspaper Nico fetch runs the full fallback chain
  including the search-grounded summary.
- `prototype-v30-np-autofetch.html`: functional reference. Newspaper
  manual add auto-fetches the Nico summary when a link is pasted; visible Fetch
  with Nico button on every newspaper card; headline-only cards can fetch via
  the search-grounded fallback.
- `prototype-v31-nico-errors.html`: latest functional reference. Nico failure
  mode fixed: raw model/API errors never reach the card; calm messages + Retry
  instead, technical detail to console only.

These are behavior/look references. The real build is the multi-module app
described in the cloud prompt; where the prompt and a prototype disagree,
the prompt wins.
