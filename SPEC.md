# DAILY GOODS HUB - Build Spec
For Claude Code. Read this whole file before writing any code. Do not write app code for anything else in this repo tree, and do not change any existing project.

## 0. What this is

A static hub website for THE DAILY GOODS, Jay Brody's nationally syndicated weekday radio show on SiriusXM Canada (3-5pm ET). The show team currently builds each day's rundown in a Google Sheet. This hub replaces that workflow: story feeds come in raw, the team curates a daily newspaper, pins fleeting ideas on a corkboard, preps interviews in a simple notes area, and builds the day's six-segment rundown with draggable cards. Print view produces the studio drive sheet.

Read the companion file `research-notes.md` in this folder before building. It holds the exact feed URLs, record schemas, the proven Nico Fetcher summarization pattern, the v2 visual palette, and the real rundown anatomy verified from Jay's sheets.

### 0.1 Rules of the build
- Static site only. Plain HTML/CSS/JS, no build step, no npm. CDN libraries are allowed only when proven in this stack (SortableJS for drag and drop, as used in Story Scout Next v2).
- Hosted on GitHub Pages from a brand-new repo. Deploy pattern: push to `main` publishes (see story-scout-next `.github/workflows/deploy.yml` for the established pattern; add `.nojekyll`).
- No backend. No auth. No API keys in the repo. The Gemini key is entered by Jay in the Settings page and stored in the browser's localStorage only; it never touches the repo, exports, or snapshots.
- The `daily-goods-rundowns` repo (production Story Scout Arcade + Nico Fetcher) is FROZEN. The hub may only READ from it over the network. Never write to it, never suggest changes to it, never link to anything that modifies it.
- Do not touch the story-scout-next repo or the daily-goods-rundowns repo at all. This is a separate repo.
- No sending, posting, or emailing in Jay's name anywhere in this app. Everything is local prep work.
- UI copy rule: no em dashes anywhere in the interface. Keep copy short, plain, and where it will be read on air, written to be spoken aloud.

## 1. Product overview: the seven pages

Top-level nav with six sections. Single-page app with tab navigation is preferred (one `index.html`, sections toggled in JS). State persists in localStorage (see section 3).

### 1a. Feeds (Story Scout outputs, raw, read-only)
One unified feed. The reader NEVER sees which scout a story came from: no source panels, no source labels, no source badges anywhere on this page. Origin stays in the data model only (it feeds the duplicate filter).
- Fetch both JSONs in parallel on first visit and on Refresh (permissive CORS on raw.githubusercontent.com). Loading state ("Fetching scout feeds...") while the multi-MB files load. In-memory session cache so tab switches do not refetch.
- Normalize each record to {headline, url, source, category, bullets[], angle, date, slot, score}. Take the most recent `date` across BOTH sources. DEDUPE, never merge-with-badges: same URL, or same normalized headline (lowercase, strip punctuation/whitespace), appears exactly once.
- 5-day rolling history: every successful fetch upserts the latest scout day's deduped stories under that scout date in the store; keep the 5 most recent scout dates, prune older. Render day groups newest-first with headers ("Friday, Oct 16 (47 stories)"). Same-day refreshes update in place, never duplicate. Cap: if one day exceeds ~300 stories, render the top 150 by score and note the cap.
- Sunday noon refresh (America/Toronto): when the most recent Sunday 12:00 PM Toronto time is newer than the last wipe marker, clear feed history AND saved stories AND tombstones, then set the marker. During the week the feed populates by day (5-day rolling history). After the Sunday wipe, a calm empty state ("Feed cleared Sunday afternoon. Fresh stories start Monday.") with no fetching until Monday. Monday starts a fresh 5-day accumulation. A missed Sunday wipe still runs on the next load.
- Each card expands (headline, bullets, URL, actions). Three actions per card:
  1. "Add to Today's Show": reveals a segment dropdown with the six segments and time slots; picking one creates the card in that builder lane immediately.
  2. "Save for Tomorrow": pins the story into its own section at the bottom of the tab; saved stories survive every refresh.
  3. "Delete": removes the story and tombstones it (by URL or normalized headline) so refreshes never resurrect it; a "Restore deleted" control lists tombstoned stories with restore buttons.
- Every card also has "Send to newspaper" (copies into the Newspaper as a scout item, deduped by URL).
- Failure: per-source inline error with a Retry button; the tab never renders empty without an explanation. The feeds never block the rest of the app.
- This page never edits story data. Raw means raw.

### 1b. Daily newspaper (curation for review)
The working shortlist for today's show. Stories arrive via "Send to newspaper" from the Feeds or the Corkboard (deduped by URL), or the manual "Add a story" composer (headline + URL).
- Working-date switcher at the top; each date keeps its own list, persisted in localStorage per day.
- Item: headline (editable), source label, URL, category tag, a small "source feed" badge (NEXT / JSTATION / MANUAL), with full card details like the builder (bullets, audio cue, Nico fetch, producer notes).
- Actions per item: drag to reorder, a segment picker (dropdown of the six segments), "Send to builder" (creates the card in the picked lane; prompts if no segment is picked), delete.
- "Send all to builder" bulk action: respects each item's picked segment; items with no segment picked go to 4:00 TRENDING TODAY.
- "Start new day" archives the current list and begins a fresh one. Archived newspapers are viewable in a simple archive list.

### 1c. Interviews (cards, per Jay 2026-10-19)
Each upcoming interview is a full card that looks and behaves like a builder card: collapsed row (guest name, slot badge, format badge, show date), click to expand, up/down arrows to reorder (persisted), delete with confirm.
- Expanded card fields, in order: Guest name, Show date, Slot (3:20 / 4:20 select), Format (in-studio / zoom / pre-tape / hold select), INTRO (textarea), SEGMENT INFO (textarea), TAPE OUT (text input, labeled "HOST speaks last"), Producer notes (textarea), Questions (single textarea, one question per line, with a "+ Add question" button that appends a line).
- "Push to today's show" on every card: pushes into TODAY's builder, sets the matching slot lane (3:20 or 4:20) to Interview type (talk payload preserved), and fills the guest block (name, slot, INTRO, SEGMENT INFO, TAPE OUT) as one undo entry. Locked target lane: calm "That segment is locked." message, no change.
- Add interview form: name, date, slot, format. Archive toggle moves past interviews to an archived list. Delete with confirm. No question generation, no prep templates, no automation beyond the push.
- Guest segments in the builder also offer a "pull from interviews" picker that copies the guest name and fills empty INTRO / SEGMENT INFO fields from the interview card.

### 1d. Rundown builder (the centerpiece)
The daily show laid out as six segment lanes in air order (see section 2). Cards are dragged between lanes and reordered inside them. Print produces the drive sheet.
- Today | Tomorrow day switcher in the builder header: each day holds its own autosaved rundown (lanes, cards, locks, furniture, Today-on-the-Show text); every story card has a Push to Tomorrow action that moves it into the same segment in tomorrow's rundown (undo-able). Post-show wrap: a "Wrap up" button on Today's view opens a modal triaging every topic as Aired, Tomorrow (push to tomorrow's same segment), or Delete; one Apply runs the whole batch as a single undo entry.
- Segment naming: 3:05 THE LIST and 4:00 TRENDING TODAY are fixed anchors, never renamable. The 4:40 slot auto-names by the viewed day's Toronto weekday: WHAT THE FRIDAY on Fridays, KICKER/GOODBYES any other day. The 3:20, 3:40, and 4:20 lanes are click-to-rename per day (defaults GUEST / LIFESTYLE CHAT / GUEST); the effective name flows into lane headers, the Today on the Show auto-lineup, print, and print preview.
- Layout modes: a "Standard | Grid" toggle in the builder header, persisted. STANDARD is the default and mirrors the Google Sheets show-notes anatomy: lanes stack vertically full-width in air order; each segment opens with a time/label row ("3:05 - THE LIST"), then label|content rows: "SEGMENT N: INFO" holding the cards rendered as "TOPIC 1:", "TOPIC 2:"... (backup-flagged cards render as "BACK UP TOPIC:", visually distinct), then a TEASE row per segment built as locked scaffolding: segments 1/2/4/5 show locked `[HOST TEASE / THROW TO BREAK]`, an editable "UP NEXT --" field (multi-line tease copy), and locked `**BACK IN 2 MINUTES // 120 SECONDS**`; segment 3 swaps the closer for locked `**HARD OUT AT 3:57**`; segment 6 shows locked `[HOST TEASE TO TOMORROW / GOODBYES]`, an editable "TOMORROW --" field, locked `UP NEXT -- THE WRAP UP!`, and locked `**HARD OUT AT 4:57**`. Tease content is per-day; the locked lines are constants. Tease blocks render in print and the print preview for every segment including interview segments. Guest lanes render as "GUEST:" (time - name - topic), "INTRO:", "SEGMENT N: INFO", "TAPE OUT (host speaks last):", "TEASE:" rows. A "HOUR 2" divider sits between segments 3 and 4.
- Fixed furniture, present every day, never draggable, never deletable (but editable, content changes daily): the header block (show title + date, Zoom link, studio listener/guest lines, TODAY ON THE SHOW auto-built live from the six lanes' titles + guest names/topics, SHOW INTRO with the two guests and CHATS 1-3) and the footer block (TEASE TO TOMORROW/THIS WEEK outro block, PUSH with the show email). Furniture stays locked in place; only segment INFO areas hold draggable cards.
- In-app print preview: a "Preview drive sheet" button opens a read-only modal rendering the drive sheet exactly as it will print (black on white, full anatomy from live state), with its own Print button; closes via X, Esc, or click-outside.
- GRID keeps the compact multi-column lane layout. All interactions work identically in both modes.
- Every story card and guest block has a "Copy" button that writes plain text formatted for pasting straight into Google Sheets (each line becomes a row in the active column): story cards copy HEADLINE (`[BACKUP] ` prefix when backup-flagged), `PRODUCER NOTE: <note>` only if present, one line per talking point prefixed "• ", `Source: <label>` and URL lines, `AUDIO CUE: <cue>` only if present; guest blocks copy guest name + slot, INTRO paragraphs, `SEGMENT INFO:`, `TAPE OUT:` (host speaks last), `TEASE:` lines, skipping empties. Toast "Copied for Sheets" on success.

### 1e. Corkboard (v1: Jay's corkboard-and-post-its, digital)
A freeform board for fleeting ideas, one-liners, and notes that are not stories yet. Styled like post-it notes on a corkboard: small colored notes, drag to arrange freely, always a satisfying little tilt and shadow.
- Quick-add: type and the note lands on the board. Notes have a short title/body (plain text with spellcheck), a color picker (4-5 colors), and drag-to-arrange positioning (free placement within the board, persisted).
- Every note has "Send to newspaper" (becomes a MANUAL newspaper item) and delete. Notes are lightweight on purpose: no rich text, no metadata.
- This replaces the CHAT BRAINSTORM tab's job. It lives as its own nav tab so the builder page stays focused.

### 1f. The Keep (works like the actual Google Keep app, revised 2026-10-19)
A persistent home for loose notes and longer writing: the hub's private Google Keep. Distinct from the Corkboard (fleeting spatial post-its with send-to-newspaper); The Keep is where notes live long-term. Jay's directive: it must work like the real Keep app, with ONE long-form box for notes, not card-based editing. Think long-form notepad: ease of use is the priority, opening a note feels like opening a notepad, zero friction.
- The primary composer is a single "Take a note..." long-form box at the top. Clicking it expands into title + one continuous auto-expanding body field: plain text with spellcheck, one writing surface.
- Clicking any existing note opens the SAME long-form box editor (title on top, body as one long scrolling field; sheet/notepad feel, not a card overlay).
- Notes browse in Keep's familiar masonry grid (cards are fine for display; the long-form box is for writing and editing).
- Any note toggles between note and checklist mode (checkbox items, checked items sink, hide-checked option).
- Per-note color backgrounds (Keep's soft palette, ~8 tints).
- Labels: create labels, assign multiple per note, filter the grid by label from a sidebar.
- Pin to top: pinned notes sit in their own section above the rest.
- Archive and Trash: archived notes leave the grid but stay searchable; trashed notes auto-delete after 30 days, with manual empty-trash.
- Search across titles and bodies. Everything autosaves per section 3.1.

### 1h. Tools & Resources (directory of everything Jay has built)
A tab with stylized cards linking out to Jay's apps and the tools/software he uses. Three defaults ship pre-loaded: The JayStation (his original story-prep app), Story Scout Next (the playground for the next generation), and the Nico Fetcher (the standalone AI summarizer). Each card shows a short explanation and an Open button (new tab). An "Add a tool" composer (name, URL, explanation, category: App / Tool / Software / Resource) plus per-card edit and delete, all persisted in localStorage; deleting a default keeps it deleted.
### 1i. Settings page
Gear icon in the app header opens Settings. Holds the API key management above (masked field, show/hide, copy, test against the models endpoint, clear). Settings may grow (Zoom/studio defaults already live in the builder furniture); keep it a single page.

### 1g. Archive (automatic, mirrors the Drive; headline deliverable, revised 2026-10-19)
Every finished show rundown is stored here, automatically. No manual archiving step exists: locking in all six lanes of a day creates that day's archive entry (snapshot of rundown + newspaper + corkboard, with `lockedAt` timestamp). Unlocking later never deletes the entry; re-locking updates it.
- Month groups (October 2026, September 2026, ...) mirroring the Drive's month tabs. Inside each month, one entry per show day, labeled with the date.
- Clicking a day opens the full rundown read-only, in the same layout as the print drive sheet. Actions per day: Print, Copy as text, Download JSON, Download Markdown.
- THE SHOW DATASET: the archive is where the fine-tuning dataset lives and grows. Every publish saves `YYYY-MM-DD.json` (full RundownDay: segments, cards, headlines, bullets, audio cues, producer notes, guest blocks, teases) and `YYYY-MM-DD.md` (clean markdown of the same) as a side effect of the normal workflow. A "Download dataset" action at the top exports the whole collection. Over time this becomes a complete corpus of finished Daily Goods rundowns for training and analysis. Dataset integrity matters: entries are immutable, snapshots are written at lock-in time, and nothing in the app ever rewrites history.
- Archive entries are immutable: opening an old day never edits it. A "Duplicate to today" action can copy an old day's structure into a fresh working day.

## 2. Rundown builder: layout and interactions

### 2.1 The six segments (fixed, always in this order)
1. `s1` 3:05 THE LIST (story)
2. `s2` 3:20 GUEST (guest)
3. `s3` 3:40 LIFESTYLE CHAT (story)
4. `s4` 4:00 TRENDING TODAY (story)
5. `s5` 4:20 GUEST (guest)
6. `s6` 4:40 KICKER/GOODBYES (story)

Story segments hold 3-5 topic cards plus any number of backup cards. (Trending Today holds up to 5; the others hold 3-4. Do not hard-block overflow, just let the lane grow; the host decides.)

### 2.2 Cards (topics)
A card shows: headline, source label, first URL, bullet talking points (collapsible), audio cue line, backup badge, segment assignment. Expanded card view (click to open) shows everything plus:
- All URLs (add/remove)
- Talking points as a single multi-line textarea (section 2.4): one bullet per line, plain text, spellcheck. An "add talking point" button appends a new empty line and focuses it. Empty lines are ignored when rendering.
- Audio cue field (free text, e.g. `CLIP - "don't settle"` or `clip - in: ... / out: ...`)
- "Fetch with Nico" button (section 5)
- Mark/unmark as backup
- Producer notes as a rich-text field (section 2.4, free text). Any producer notes entered always print on the drive sheet above their card's talking points (order: headline, producer note, bullets), styled distinctly (italic, prefixed "PRODUCER NOTE:"). No toggles. In the card editor the Producer notes field sits above Talking points.
- Source feed badge

### 2.3 Interactions
- Drag cards between segment lanes and reorder within a lane. Use SortableJS (CDN), the same library Story Scout Next v2 uses. Also provide up/down arrow buttons on each card as a non-drag alternative (phone friendly, reliable under time pressure). Motion is a core requirement, not decoration: dragging must feel like sliding a physical card in a solitaire game, never like a ghost floating over the page. The dragged card is the real card, fully opaque, lifted above everything (scale-up + soft shadow), tracking the pointer 1:1 with no lag. As it moves over a lane, the sibling cards part in real time to open the exact slot where it will land (live insertion preview, smoothly animated), so the drop target is obvious from the layout itself. On release the card glides from the release point into the slot and settles with a quick confirm pulse, the small "pop" confirming the lock (~200ms, then gone). The card never vanishes, never goes translucent, never detaches from the pointer. CSS transitions/keyframes only, no animation libraries. Touch behavior: drag starts on a short long-press (~200ms, via SortableJS `delay`/`delayOnTouchOnly`) so vertical scroll still works; on touch devices use `forceFallback: true` so the card itself is the drag proxy instead of the browser's native ghost image; the page must not scroll while a drag is active.
- Keyboard control (v1): the builder is fully keyboard-operable for deadline speed. `↑`/`↓` move the focused card within its lane; `Enter` confirms/locks a move; double-click (or `Enter` on a resting card) opens the expanded card; `Esc` closes it. Focus is always visible. This is additive: mouse/touch drag keeps working exactly as before.
- Expanded cards are not draggable: while a card is open for editing, drag is disabled on it (so text selection never fights with dragging) and it shows prominent up/down arrow buttons to move it within its lane. Collapsing the card restores draggability. Collapsed cards keep both drag and their smaller arrow buttons.
- Copy as text: per-segment "Copy segment" and whole-day "Copy day as text" actions put the rundown on the clipboard in clean plain text (headlines, bullets as lines, audio cues, backups marked), formatted for pasting straight into Google Sheets or the archive. No formatting chrome, no HTML.
- Backup flag per card: backups stay attached to their segment but render visually distinct (dimmed, dashed border, "BACKUP" tag). A "promote to main" action unflags them back into the main stack.
- Segment lock (v1): every lane header has a lock toggle. A locked lane is read-only: no cards dragged in or out, no reordering, no card edits, no add, fields unfocusable. Locked lanes show a small lock icon and a calmer, settled treatment. Unlocking is one click on the same toggle. "Lock all" / "Unlock all" actions sit above the lanes. EXCEPTION: the per-card Copy for Sheets button keeps working on locked lanes (story cards and guest blocks), writing the exact same format as unlocked; copying is not editing.
- Finalize day: when all six lanes are locked, the builder shows a "Show locked for [date]" state with the primary action "Print drive sheet". Locking in the full day automatically creates that day's archive entry (section 1f): snapshot of the rundown + newspaper + corkboard, timestamped. Unlocking any lane afterwards does not delete the archive entry; re-locking updates it.
- Per-card "Fetch with Nico" button summarizes the card's primary link in place: it fills in the headline (from the fetched TITLE line, only if the card headline is empty or matches a placeholder) and the bullet talking points, and sets `fetched: true`. Fetching state shows a spinner on the card; errors show inline with a retry link. See section 5 for the exact pattern.
- Story cards can be added manually (headline + URL) or from the newspaper via the segment picker.
- Every story lane has a visible "Add story" button in its lane header. Clicking it opens an inline composer right in the lane (headline + URL fields, Add/Cancel); on add, the new card appears at the end of the lane (or at the drop position if the composer is placed inline) and opens expanded for immediate editing. This is the fastest path from an idea to a card and must never be more than one click away.
- Tease field per segment (free text, prefilled with the sheet's default tease lines). Segment 3's tease defaults to the hour-1 hard out note ("HARD OUT AT 3:57"), segment 6's to the end-of-show tease ("HARD OUT AT 4:57").
- Guest segments (s2, s5) use a different card shape, a single guest block per lane: guest name, slot time, INTRO (rich text, section 2.4), SEGMENT INFO (rich text), TAPE OUT line (labeled "HOST speaks last"), plus the tease field. A guest segment holds one guest; a "pull from interviews" picker prefills name and notes.
- Header block (auto-generated, editable): show title, show date, Zoom link + password, studio listener line, studio guest line, TODAY ON THE SHOW running order (auto-built from the six segment titles plus guest names), SHOW INTRO with the two guest names and CHATS 1-3 (three short topic-name fields).
- Footer block: PUSH email, TEASE TO TOMORROW textarea with the hard-out labels.
- Header/footer defaults live in a settings area stored in localStorage (section 3). Zoom link/password and studio lines ship blank in defaults; each user pastes them once into Settings (decided 2026-10-16; they gate live show access and do not belong in a public repo).

### 2.4 Rich-text card editing (Word-like, with spellcheck)
Every substantial free-text field on a card is a small rich-text editor, not a raw textarea. It must feel like a miniature Word/Google Doc: bold, italic, underline, bullet and numbered lists, and working spellcheck with the familiar red underline. Spellcheck comes from the browser natively in editable rich-text fields; ensure `spellcheck="true"` is set and never disabled.
- Applies to: bullet talking points, producer notes, guest segment INTRO and SEGMENT INFO, tease fields, interview notes on the Interviews page.
- Link handling: any URL pasted without a scheme (e.g. `nytimes.com/...`) is normalized to `https://...` at save time and defensively at render time, everywhere links appear (builder cards, newspaper, feeds, tools, Zoom link). Links never resolve relative to the hub URL.
- Suggested implementation: Quill 2.x via CDN (single script tag, no build step, stable API, proven for years). Keep the toolbar restrained: bold, italic, underline, bullet list, numbered list. This is talking points, not a document; a minimal toolbar keeps it fast and stable. Fallback if Quill proves awkward: TipTap via ESM CDN. Do not hand-build a custom editor.
- Headlines stay plain single-line inputs (with `spellcheck="true"`). URLs stay plain inputs. Audio cue stays a plain input.
- The editor must never fight the user: no auto-formatting surprises, no lag on typing, content saved to the card model on every change (debounced) so nothing is lost.
- Print output strips editor chrome and renders clean text (section 2.5).

### 2.5 Print drive sheet
A print stylesheet (`@media print`) that reproduces the rundown one-to-one with the Google Sheet layout the studio already knows:
- Header block, then the six segments in order, each with its SEGMENT N: INFO topics (headline + bullets + audio cues), backup topics marked, tease rows, guest intro/tape-out for guest segments, then the footer with push email and tomorrow's tease.
- Hide all builder chrome (nav, buttons, drag handles, fetch buttons) in print. Cards print expanded.
- Producer notes always print when present: a card with non-empty producer notes prints them directly under the card, styled distinctly (italic, prefixed "PRODUCER NOTE:").
- Page-break guidance: break after segment 3 (hour 1) so each hour lands on its own page.
- A prominent "Print drive sheet" button on the builder page that calls `window.print()`.

## 3. Data models

Persist everything in localStorage under one key: `dg-hub-v1`. The stored object carries `stateVersion: 1` and the collections below. On load, if the stored version mismatches, keep the old blob under a backup key and start fresh (never silently migrate destructive changes).

```js
StoryCard {
  id: string,            // uuid
  headline: string,
  urls: string[],        // primary link first
  bullets: string[],     // talking points (Nico-populated); plain-text strings, one per line in the single editor textarea
  audioCue: string,      // e.g. CLIP labels with in/out notes
  isBackup: boolean,
  order: number,         // position inside its segment
  segmentId: "s1"|"s2"|"s3"|"s4"|"s5"|"s6"|"newspaper",
  sourceLabel: string,   // e.g. "Variety"
  category: string,      // e.g. "ENTERTAINMENT"
  angle: string,         // scout-written angle, optional
  producerNotes: string, // rich-text HTML, free text; always prints when non-empty
  origin: { feed: "scout-next"|"jstation"|"manual"|"newspaper", storyId?: string },
  fetched: boolean,      // a Nico summary has been pulled for this card
  createdAt: string, updatedAt: string  // ISO
}

Segment {
  id: "s1".."s6",
  kind: "story"|"guest",
  time: "3:05"|"3:20"|"3:40"|"4:00"|"4:20"|"4:40",
  title: string,         // e.g. "THE LIST"
  tease: string,
  // guest segments only:
  guestName: string, guestIntro: string, segmentInfo: string, tapeOut: string,
  interviewId?: string,  // link back to the Interviews page entry
  cards: StoryCard[]     // story segments only
}

RundownDay {
  id: string,            // "2026-10-17"
  date: string,          // show date
  chats: string[3],      // show intro chats 1-3
  tomorrowTease: string,
  segments: Segment[6],
  createdAt: string, updatedAt: string
}

Interview {
  id: string,
  name: string,
  showDate: string,
  slot: "3:20"|"4:20",
  format: "in-studio"|"zoom"|"pre-tape"|"hold",
  notes: string,         // plain copy-paste dump
  archived: boolean,
  createdAt: string, updatedAt: string
}

Settings {
  showTitle: string,          // "THE DAILY GOODS"
  zoomLink: string, zoomPassword: string,
  studioListenerLine: string, studioGuestLine: string,
  pushEmail: string,          // TheDailyGoods@siriusxm.ca
}
```

The newspaper is stored as `RundownDay`-independent: `newspaper: { date: string, items: StoryCard[] }` where items have `segmentId: "newspaper"`. Archive entries are created automatically when a day is fully locked in: `archive: { newspapers: [...], rundowns: [...] }`, each entry carrying date, `lockedAt` timestamp, and the full JSON snapshot (rundown + newspaper + corkboard). Entries are immutable; re-locking a day updates its entry.

```js
CorkNote {
  id: string,            // uuid
  body: string,          // plain text with spellcheck
  color: string,         // one of 4-5 post-it colors
  x: number, y: number,  // free position on the board
  createdAt: string, updatedAt: string
}
```
Stored as `corkboard: { notes: CorkNote[] }`, per-day like the newspaper, included in the archive snapshots.

Settings page (gear icon in the app header, not a nav tab): API key management with an unmistakable status pill (green "API key loaded" + masked tail showing the last 4 characters, or amber "No API key saved"; masked field, show/hide, copy, test, clear; stored in localStorage only, never in the repo); Show / Studio section with Zoom link, Zoom password, listener line, and guest line (all locked read-only in the builder); edit the furniture defaults, export the whole localStorage state as a JSON file, import it back. The API key is EXCLUDED from export/import. Export/import is the backup story; label the buttons plainly.

### 3.1 Autosave and crash recovery (top priority)
A crash must never lose work. This is a headline requirement, not a nice-to-have, and it ships in milestone 1.
- Every state mutation writes synchronously to localStorage. The data for one rundown day is small (tens of KB); synchronous writes are fast enough and mean there is no "unsaved changes" state at all. No save button anywhere in the UI.
- Rich-text editor content saves on every text-change event (debounced ~500ms), so mid-sentence typing survives a crash.
- Rotating backups: keep the last 3 full state snapshots under timestamped keys (`dg-hub-v1-backup-<ts>`), written on the same mutation path. If the primary key ever fails to parse, the app automatically falls back to the newest intact backup. Corruption of one key can never nuke the day.
- Crash detection: set a `dg-hub-v1-clean-exit` flag on `beforeunload`; clear it on load. If the flag is absent at load, the previous session crashed: load the newest intact state and show a calm banner ("Recovered your work from [time]. Nothing was lost."). No user action required.
- The manual JSON export/import in Settings remains as the belt-and-suspenders backup and the way work moves between machines.
- Test: kill the tab mid-typing during QA; the next load must restore everything to the last keystroke.

## 4. Data integration

Exact sources (researched 2026-10-16; full schemas in `research-notes.md`):

- Story Scout Next: `https://raw.githubusercontent.com/jasonroti-pixel/story-scout-next/main/data/stories.json` (~6 MB, cumulative). Envelope `{generated_at, timezone, date, slot, pipeline_version, count, stories[]}`.
- J Station (original, frozen repo, read-only): `https://raw.githubusercontent.com/jasonroti-pixel/daily-goods-rundowns/main/stories.json` (~3.75 MB, plain array).

Fetch strategy:
- One `fetch()` per feed per refresh, in parallel, directly in the browser (permissive CORS on raw.githubusercontent.com). Loading state while the multi-megabyte files load. In-memory session cache so tab switches do not refetch; Refresh re-fetches.
- After fetching, normalize records, take the newest `date` across BOTH feeds, dedupe (same URL or same normalized headline: lowercase, strip punctuation/whitespace), and upsert that day's stories under its scout date in a 5-day rolling history in the store (prune to the 5 most recent scout dates; same-day refreshes update in place). Render day groups newest-first.
- Sunday noon refresh (America/Toronto): when the most recent Sunday 12:00 PM Toronto time is newer than the last wipe marker, clear feed history, saved stories, and tombstones; post-wipe shows the "cleared Sunday afternoon" empty state until Monday; Monday starts fresh.
- Manual Refresh button on the Feeds page re-fetches. Because the files are multi-megabyte, never store raw payloads in localStorage; normalized per-day stories are small enough for the store, raw payloads stay in memory only.
- Fallback when unreachable: per-source inline error with Retry; keep the last good day-groups rendering and let everything else work. The builder, newspaper, and interviews never depend on the feeds.
- Duplicate handling (v1): the same story often arrives from both scouts. Dedupe by normalized headline match (lowercase, strip punctuation/whitespace) plus same-URL match. Duplicates appear exactly once. Origin is kept in the data model only; the Feeds UI never shows which scout a story came from.

### 4.1 Decoupling contract (Jay's standing rule)
The Story Scout apps and the Nico Fetcher page are independent products. They are never modified, moved, or removed as part of this project, and the hub must never depend on their internals.
- The hub depends only on the published JSON shape of each feed (field names documented in `research-notes.md`). The scouts can change everything else freely: their UI, their pipelines, their models, their schedules.
- If a scout changes its JSON shape, the hub must not break: detect the mismatch, keep serving the last good cache, and show a plain banner ("A story feed changed its format. Showing cached stories."). Adapting the hub's parser to the new shape is a deliberate, separate update, never an emergency.
- Same rule for summarization: the hub owns its ported copy in `nico.js`. Changes to the standalone Nico Fetcher page never propagate to the hub automatically; porting an improvement over is a conscious choice.
- Concretely: all feed parsing lives in one `feeds.js` adapter module with the expected schema version noted at the top. Nothing else in the app touches raw feed payloads.

## 5. Nico Fetcher integration (per-card summarization)

Reuse the proven pattern from `summarizer.html` in the frozen `daily-goods-rundowns` repo (exact mechanics documented in `research-notes.md`). Port the logic into a `nico.js` module in this repo; do not hotlink the other repo's JS.

- Gemini API key: pre-installed in the hub. Jay approved this for the trusted-only beta: anyone on the site just clicks the Fetch with Nico button and it works, zero key setup. Put the key in a clearly marked `config.js` as `const GEMINI_API_KEY = "..."`, the single config constant the whole app reads. Mitigation, required: restrict the key in Google AI Studio to this site's URL via HTTP referrer restrictions. There is no key-entry UI in v1. Per-user keys can come in a future update if the hub ever goes wider than the trusted beta.
- Article fetch: primary path is Gemini's `url_context` tool reading the card's primary URL, guarded by the `urlContextMetadata` SUCCESS check. X/Twitter special case: for x.com/twitter.com URLs, try the fxtwitter API (`https://api.fxtwitter.com/<user>/status/<id>`) for the tweet text first, then summarize that. Fallback path: fetch the article text in-browser via Jina Reader (`https://r.jina.ai/<url>`), then the CORS proxy chain (`api.allorigins.win`, `corsproxy.io`) with Readability extraction, then summarize the fetched text. Keep the Reddit special-case handling from the original. Last resort, never the default: if every fetch path fails, Google the story: call Gemini with the `google_search` grounding tool using the card's headline as the query, and summarize from the grounded results. When the Google fallback produces the bullets, the card shows a small "via Google search" badge so the host knows the source. Every network call in the chain carries a timeout (fxtwitter 20s, Jina 30s, each Gemini call 60s, model discovery 20s); a timeout falls through to the next step like any other failure, so a fetch can never stall for minutes.
- Model selection: at runtime list models and rank the flash family (flash-lite first for the biggest free daily quota), but ALWAYS try a known-good modern flash model first (currently `gemini-3.8-flash`; older `gemini-2.5-*` names 404 for newer keys). Remember the last working model. No raw model or API error text is ever shown on a card: the chain silently skips unavailable models, and if every model fails the card shows a calm message ("Nico couldn't reach the AI models just now." or "Nico couldn't summarize this story.") with a Retry link; technical detail goes to the console only. Generation config: temperature 0.25, maxOutputTokens 3072, thinkingBudget 0, retry without thinkingConfig on rejection.
- Prompt: reuse the original STYLE_RULES verbatim and the `TITLE: <headline>` plus `- ` bullets output contract. Parse into headline + bullets (max 10).
- On "Fetch with Nico": set the card to fetching state, run the pipeline for the card's primary URL, then write headline (only if empty or a placeholder) and bullets into the card, set `fetched: true`, and re-render. Each bullet gets a delete X in the expanded card. Surface errors inline on the card with a retry action; never throw a modal.
- Cost control: no shared quota concerns beyond the free tier; the fetch button is explicit per card so usage stays intentional.
- Independence: the standalone Nico Fetcher page is untouched by this project and stays exactly where it is. The hub's `nico.js` is a port, not a link: improving the fetcher page never changes the hub, and improving the hub's summarizer never changes the fetcher page.

## 6. Tech constraints
- Static GitHub Pages site, brand-new repo. Plain HTML/CSS/JS, no build step. CDN libs only when proven (SortableJS for drag and drop; Quill 2.x for rich-text card editing per section 2.4; Readability bundle for the fetcher fallback, copied from the same source the Nico Fetcher page uses).
- No backend, no auth, no tracking, no analytics.
- localStorage-first persistence (section 3). Export/import JSON for backup.
- FROZEN repo rule: never write to `daily-goods-rundowns`, never propose changes to it. Read-only network fetches of its data are fine.
- Keep the story-scout-next repo untouched as well. The hub reads its public data files only.

### 6.1 Code architecture: built to be changed (Jay's standing rule)
The app will grow. Functionality and look must stay easy to change at any time, by Jay or any future developer, with no tribal knowledge required.
- One module per concern, plain `<script>` files, no bundler: `store.js` (all state + autosave, the only writer), `feeds.js` (feed adapters per section 4.1), `nico.js` (summarization per section 5), one render module per page (`feeds-page.js`, `newspaper-page.js`, `corkboard-page.js`, `interviews-page.js`, `builder-page.js`), `export.js` (Google Doc + dataset export per section 10).
- One-way data flow: UI events call store mutations, the store autosaves synchronously, then re-renders. No module reaches into another module's DOM. No global variables except the store and config.
- The entire look lives in `theme.css` as CSS custom properties: every color, font, spacing, radius, and shadow is a variable. Changing the visual design means editing that one file. No inline styles anywhere, no color or font literals in JS.
- Semantic class names, clear function names, comments on anything non-obvious. A `README.md` in the repo documents the architecture: where state lives, how to add a page or a segment, how to retheme, and the feed contract from section 4.1.
- Stability rule: new features add modules; they never rewrite existing ones. The autosave path in `store.js` is the most protected code in the repo: changes to it require re-running the crash test from section 3.1.

## 7. Visual and motion direction
- Abandon the Story Scout arcade motif entirely. This is a professional production tool: modern, calm, beautiful, and above all stable. No pixel art, no arcade palette, no neon.
- Aesthetic (LOCKED 2026-10-17, prototype v15 "Royal Signal"): royal blue, confident and saturated. Color lives in the background, not just the accents. Professional production tool: modern, beautiful, stable at a glance on a studio screen under time pressure. No arcade motif, no pixel art, no neon.
- Palette, the user's four colors only: Marker blue #8DBBFF, Warm gray #F6F5F1, Linear indigo #5E6AD2, Profound blue #376CFF. App background is a blue field: marker-blue wash at top melting through warm gray into a linear-indigo tint at the bottom. The app header is a saturated blue gradient with white text: the darkest blue (Linear indigo #5E6AD2) behind the title, Profound blue #376CFF at the other end. Never ivory in the header. Segment lanes are deep indigo panels holding white cards with 1px hairline borders; dragged cards lift with a blue glow shadow. Primary accent is Profound blue (CTAs, active tab, primary buttons as a profound-blue gradient with white text); Linear indigo secondary; Marker blue as highlighter (expanded-headline wash, just-added flash, focus rings). An accent switcher offers Profound / Indigo / Marker, each AAA-checked. Status colors (backup flags, alerts) are saturated hues chosen for instant legibility, never pastels. Dark mode: deep indigo-navy (#12122B) base with marker-blue glow blooms, brightened blue accents, header stays a saturated blue gradient (never pale). Print drive sheet stays black on white regardless of theme. The accent lives in one CSS custom property (`--accent`) so the app can be re-skinned in an afternoon.
- Typography: Fraunces (serif, headlines and brand, italics welcome) for display; Geist for UI/body; Space Mono for timecodes, labels, and badges, all via Google Fonts (open license, CDN). Body at 14-15px minimum; lane headers in Fraunces 600 with tight tracking. Enable tabular numerals (`font-feature-settings: "tnum"`) on every time field so segment timings align.
- Motion is a first-class requirement: everything the user touches should feel fluid and satisfying, like sliding a physical card in a solitaire game. Concrete recipe: duration scale 100ms hovers, 150ms toggles, 250ms default, 400ms panel slides, never over 600ms; easing `cubic-bezier(0.16,1,0.3,1)` for entering. On drag start the card lifts: it stays fully opaque (the real card, never a translucent ghost), renders above everything, `scale(1.04)` with a deepened soft shadow, 150ms ease-out, and tracks the pointer 1:1 via GPU transforms with no perceptible lag. While dragging over a lane, sibling cards part in real time to open the exact slot where the card will land (live insertion preview, animated with the same easing); the drop target reads from the layout itself. A subtle drop-indicator line may reinforce the slot but the parting siblings are the primary cue. On release the card glides from the release point into the slot (FLIP-style), SortableJS `animation: 220` settles it, then a one-shot ~300ms confirm pulse (scale 1 to 1.02 to 1 plus a fading accent ring), the small "pop" confirming the lock. The card never vanishes, never goes translucent, never detaches from the pointer at any point. Touch: drag engages on a short long-press (~200ms via `delay`/`delayOnTouchOnly`) so lane scrolling still works; `forceFallback: true` on touch so the card itself is the drag proxy instead of the browser's native ghost image; page scroll is locked while a drag is active. Lane entrances stagger 30-50ms, capped at 300ms total. Always honor `prefers-reduced-motion`. No bouncy springs; stable beats clever.
- Interaction grammar to borrow: hover-revealed drag handles per card (fade in, calm at rest), progressive disclosure inside cards (headline + key metadata at rest; summary, cues, notes behind expansion), generous whitespace so lanes stay calm at 2:55pm.
- Implementation: CSS transitions and keyframe animations only. No animation libraries. Stability beats cleverness. SortableJS stays for drag and drop (proven in this stack).
- UI copy: no em dashes anywhere. Short, plain labels. Copy that lands in briefs is written to be spoken aloud.

## 8. Build milestones (in this order)
1. Static shell + tab nav + settings, and the segment board with all six lanes, SortableJS drag and drop plus up/down buttons, backup flagging. Autosave and crash recovery per section 3.1 (synchronous writes, rotating backups, crash banner) ship in this milestone and are tested by killing the tab mid-typing. LocalStorage persistence for one working rundown day.
2. Card model + expanded card + per-card Nico fetch (`nico.js` + `config.js` with the pre-installed key), bullet delete, audio cue field, header/footer furniture fields, guest segment shape.
3. Newspaper curation flow (promote from feeds, manual add, reorder, segment picker into the builder, per-day with archive) plus the Corkboard (post-it notes, drag-to-arrange, send-to-newspaper).
4. Interviews page (simple list + expandable notes dump + archive, exactly as scoped in 1c).
5. Print drive sheet stylesheet matching the sheet layout, with the Print button and page-break rules; segment lock/unlock + lock-all/unlock-all; finalize-day flow (all locked -> "Print drive sheet" primary action + automatic archive entry); Archive tab (month groups, read-only day view, print/copy/download per day, dataset download).
6. Feed integrations (both panels, latest-run filtering, caching, refresh, fallbacks) plus duplicate merge across the two scouts.
7. Google Doc export: "Publish to Google Doc" via the Docs API (OAuth Client ID in config.js), plus JSON + Markdown dataset snapshots per published day and a dataset download action. See section 10.
8. Keyboard control for the builder (arrows/Enter/Esc/double-click per section 2.3) plus per-segment and whole-day "Copy as text" actions.

Do not start milestone 2 until milestone 1 works end to end in the browser. Test each milestone against the real rundown anatomy: build one full day that looks like the October 2nd show from the sheets.

## 9. Explicitly out of scope
- Changing Story Scout Next or the original J Station app in any way.
- Touching the frozen `daily-goods-rundowns` repo.
- Any backend, auth, user accounts, or server-side storage.
- Anything that sends, posts, or emails in Jay's name.
- Interview prep automation, question generation, or templates on the Interviews page (explicitly simplified by Jay).
- Social publishing, analytics, or audience features.

## 10. Daily Google Doc export and archive dataset
The Google Sheets are retired as working copies, but the daily rundown document lives on: every show day ends as a Google Doc, and every finalized day also saves a structured snapshot that doubles as a fine-tuning dataset.
- "Publish to Google Doc" button on the builder. It creates (or updates) a dated doc, e.g. `THE DAILY GOODS - Friday, October 17, 2026`, in a configured Drive folder, containing the full rundown in the sheet's text layout (header, six segments with topics/bullets/audio cues/backups, teases, guest intros, footer). This is the human-readable archive that replaces the daily sheet.
- Implementation for a static site: Google Docs API via the `gapi` client library with an OAuth 2.0 Client ID. Jay creates the Client ID once in Google Cloud Console (free) and adds it to `config.js` next to the Gemini key. The user signs in with Google once per session; the token lives in memory only, never in localStorage. First version may alternatively offer "Download as .docx" (via the `docx` CDN library) and "Copy as rich text" as fallbacks; the direct-to-Drive publish is the target.
- Dataset snapshot: every publish also saves two files to the local archive: `YYYY-MM-DD.json` (the full RundownDay object, section 3) and `YYYY-MM-DD.md` (clean markdown of the rundown text). Structured, consistent, dated: this is the fine-tuning dataset, generated as a side effect of the normal workflow at zero extra effort. Include a "Download dataset" action that zips the archive.
- Naming and folder convention mirrors his current tabs: one folder per month, one doc per show day. Docs are never working copies; all editing happens in the hub.

## 11. Future upgrade ideas (status per Jay's review, 2026-10-16)
Do not build these in v1 unless marked otherwise. Each carries its decided status and, where Jay asked, the implementation approach written up now so it can be built later without re-thinking.

- Segment-fit scoring: DROPPED. The two scouts already categorize stories themselves; no hub work needed.
- Draft the day: CONFIRMED FUTURE (not v1). One click proposes a full six-segment rundown from the newspaper; the host rearranges from a draft instead of from zero.
- Tease writer: APPROACH SPECED, BUILD DEFERRED. Per segment, a "Draft tease" button sends that segment's headlines to Gemini (same pre-installed key, short prompt: "write a 1-2 line spoken host tease throwing to break for these topics, punchy, no em dashes") and fills the tease field; the host edits freely. Same pattern for tomorrow's tease from the whole day. No automation runs without the button press.
- Talkability flag: DOCUMENTED FOR FUTURE. Automatic pass over each story against his editorial bar ("would a Canadian talk-radio host open with this?"), flagging bleak wire-news items. Approach TBD at build time.
- Duplicate merge: IN V1 (section 4). Decided.
- Already-aired check: FUTURE, via a new Flags page. Design now: a "Flags" nav section with a count badge, listing warnings across the day's stories. First flag type is already-aired: fuzzy headline match against the last 7 days of published rundown archives, showing "Ran on [date]" with a link to that day. Dismissible per flag. Other flag types (stale story, missing source link) can join later. Not on the story-building pages; it lives in the main nav.
- Freshness + Canadian-angle tags: APPROACH SPECED, BUILD DEFERRED. Freshness: compute story age from the feed's date/slot vs the show date; stories older than 48h get a muted "2d old" tag. Canadian angle: keyword match (Canada, Canadian, Toronto, Ontario, Ottawa, PM, premier, etc.) plus the scout's own `canadian_angle` field when present; matches get a small maple-tag. Both are display tags only, never filters.
- Logistics strip: DEFERRED, EXPLORE LATER. Today's guests with format badges (in-studio / zoom / pre-tape / hold), pre-tape times, and prep owner (Jay/Liza), doing the guest-schedule tab's job. Data source is Outlook email + calendar, so integration needs a real design pass (likely Microsoft Graph via a small backend or a manual export flow; not a static-site freebie). Prepare the approach separately when Jay asks.
- Brainstorm inbox: BECAME THE CORKBOARD, IN V1 (section 1e). Decided.
- Show intro helper: FUTURE. Drafts CHATS 1-3 from the day's top stories; approach TBD at build time.
- Keyboard shortcuts: IN V1 (section 2.3). Decided, scoped to arrows/Enter/Esc/double-click plus copy-as-text.

## 12. Open questions (for Jay)
1. RESOLVED 2026-10-16: Zoom link/password and studio lines ship blank in defaults; each user pastes them once into Settings (localStorage only).
2. Feed files are 3.75-6 MB per fetch. The spec caches them client-side with a manual refresh. Is that acceptable, or should the scout pipelines later publish a small latest-run-only file? (That second option is out of scope for this build.)
3. Should the hub keep an archive of past rundown days, or only the current working day? The spec assumes a lightweight multi-day archive for both rundowns and newspapers.
