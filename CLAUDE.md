# CLAUDE.md

Guidance for working in this repository.

## What this is

A **single-page, mobile-first, Hebrew-RTL event website** for **מקושר בכל 15** — the
15th annual Lubavitch family conference by התאחדות החסידים ליובאוויטש.
Event: יום שלישי ז׳ מנחם־אב · 21.7.2026 · מרכז הכנסים הבינלאומי, אשקלון.
Its single job is to get an avreich to register on Tickchak and let him plan his day.

It was **implemented from a Claude Design prototype** (`מקושר בכל 15.dc.html`). That
prototype depended on the Claude Design runtime (`support.js`, `<sc-for>`/`<sc-if>`/`{{ }}`);
this repo is the **standalone production version** — plain static files, **no framework,
no build step, no runtime dependencies**.

## Layout

```
mekushar-website/
├── index.html      ← the entire site: HTML skeleton + inline <style> + one <script> renderer
├── data/
│   └── event.json  ← SINGLE SOURCE OF TRUTH — all content lives here
├── assets/
│   ├── brand/      ← logo, Rebbe photos, watercolor trees, grass strip, illustrations
│   ├── speakers/   ← speaker headshots; filename = speaker `id` in event.json
│   └── gallery/    ← photos from previous conferences
├── README.md       ← run / deploy / asset-status notes
└── CLAUDE.md       ← this file
```

There is no `package.json`, bundler, or CI. Editing `index.html` or `data/event.json` and
reloading is the whole dev loop.

## Running it

The page loads content with `fetch('data/event.json')`, and browsers block `fetch` over
`file://`. **Serve over HTTP** — do not double-click `index.html`:

```
cd <this folder>
npx serve            # or: python -m http.server 8000   or: php -S localhost:8000
```

If opened via `file://` the page shows a friendly "run a local server" overlay instead of
silently failing.

## How rendering works

`index.html` is one IIFE. Flow: `boot()` → `fetch('data/event.json')` → `render(DATA)` →
builds the whole page as an HTML string and sets `#app.innerHTML` → `wireUp(DATA)` attaches
interactivity. On fetch failure, `showError()` renders the overlay.

**Content is data-driven.** Every fact, name, time, and quote comes from `event.json`.
**Changing a session title / time / bio / FAQ answer in `event.json` changes the page.**
The ONLY hardcoded Hebrew in `index.html` is UI microcopy (countdown labels ימים/שעות/דקות,
"שים לב", "עד פתיחת הכנס", "מצטרפים השנה?", "איך מגיעים", accordion `+`, etc.). Do not add
content strings to `index.html` — add them to `event.json` and read them in a renderer.

### Code map inside index.html
- **Helpers:** `esc` (HTML-escape — use for ALL interpolated text/attrs), `pad`, `byId`,
  `initials` (strips honorific prefixes → first letter, for photo-less avatars),
  `mapsUrl`/`wazeUrl` (from `venue.coordinates`), `downloadICS`, `shapeSession`
  (joins a schedule session to its speaker/hall/track).
- **Section renderers** (each returns an HTML string): `renderHeader`, `renderHero`,
  `renderCountdownInner`, `renderAbout`, `renderTracks`, `renderSessionCard`, `renderKeynote`,
  `renderSessionsBlock`, `renderDivider`, `renderSchedule`, `renderBanquet`, `renderSpeakers`,
  `renderTestimonials`, `renderRebbeQuotes`, `renderGallery`, `renderFaq`, `renderGettingThere`,
  `renderRegister`, `renderSponsors`, `renderFooter`, `renderStickyCta`.
- **`wireUp`** wires: the live countdown (`setInterval`, updates `#mk-countdown` in place),
  the testimonials carousel (in-place text swap, prev/next/dots), the FAQ accordion
  (delegated click, toggles `hidden` + `aria-expanded`), the sticky mobile CTA
  (scroll listener toggles `.mk-show` past 640px), the ICS download button, and a
  **capture-phase `error` listener** that hides any image that fails to load (and collapses
  the hero logo card) — this is why missing brand images don't show broken icons.

### event.json shape
`event` (name/edition/subtitle/date/hours/venue/getting_there/organizers/registration/contact/
`religious_framing`/dedication), `site` (footer_credit/countdown_target/event_end/suggested_nav),
`halls[]`, `tracks[]`, `speakers[]` (`photo:null` → renders a teal initials avatar),
`schedule[]`, `quotes[]` (by `id`: hayom-yom, basi-legani, toldos-5752, azaria-magazine),
`testimonials[]`, `faq{items[]}`, `about`, `gallery`, `sponsors[]`, `perks[]`.

**Schedule `type` values** and how they render:
| type | rendering |
|---|---|
| `info` / `optional` | slim "pre-rows" above 10:00 (gates 09:00, optional 08:30 shacharit), sorted by time |
| `plenary` / `parallel` | session-card block with the giant sticky time numeral; `parallel` = up-to-4-across grid |
| `keynote` | full dark hero card (13:00) with the Azaria pull-quote |
| `break` / `meal` | slim dashed-divider row with a time range |
| `banquet` | its own orange full-bleed section (17:00) |
| `farbrengen` | parchment/globe card at the end of the banquet section (19:00) |

Relationships: `schedule[].sessions[].speaker_id → speakers[].id`, `.hall_id → halls[].id`,
`.track_id → tracks[].id`; `speakers[].photo` / `gallery.images[]` → paths under `assets/`.
The numeral colour alternates orange/teal per session block (`keynote` consumes a parity slot
even though it renders separately — keep that if you touch `renderSchedule`).

## Conventions & gotchas (read before editing CSS/layout)

- **RTL is load-bearing.** `<html lang="he" dir="rtl">`. Use logical CSS props
  (`inset-inline-*`, `margin-inline-*`, `text-align:start/end`) — never left/right.
- **Grid overflow:** all multi-column grids use `minmax(0,1fr)` tracks (NOT `1fr`) and
  `min-width:0`, otherwise long non-wrapping Hebrew content blows out the track and causes
  horizontal overflow. `body` also has `overflow-x:clip` as a backstop. If you add a grid,
  use `minmax(0,1fr)`. Sanity check after layout changes:
  `document.documentElement.scrollWidth - clientWidth` must be `0`.
- **Session cards:** the hall-name pill is a standalone ribbon at the TOP of each card, and
  the giant time numeral sits in a `~195px` sidebar with a `clamp(48px,9vw,78px)` numeral —
  these two widths are tuned so 4 cards + numeral fit without squeezing titles or overlapping.
  If you enlarge the numeral, widen `.mk-numwrap` to match or cards get cramped.
- **Motion:** entrance/reveal animations are CSS scroll-driven (`animation-timeline: view()`
  on `[data-reveal]`, `.mk-rise` on hero) and are wrapped in
  `@media (prefers-reduced-motion: no-preference)` — reduced-motion users see everything
  statically. Keep that guard on any new animation.
- **Always `esc()`** interpolated values when building HTML strings.

## Community-critical requirements (do NOT change casually)

- Religious framing must stay **verbatim** as in `event.json`: honorifics (הרבי מלך המשיח,
  שליט״א מה״מ, כ״ק אד״ש, ע״ה, יבדלחט״א, שי׳), the `ב״ה` at the top corner, the yechi line and
  the dedication (לעילוי נשמת) in the footer. Do not "modernize" or trim them.
- The Rebbe's photos are treated with dignity (never cropped through the face, never heavy
  filters, never a repeating background).
- **Footer credit is required by the site owner** and must remain, exact text, linked:
  `נבנה באהבה ע״י שגיא טוויל · ייעוץ והטמעת AI` → https://x-technologies.co.il/ (new tab, noopener).
- **No invented content.** Ticket prices are deliberately absent — Tickchak owns them; every
  CTA links to `https://tickchak.co.il/106158?ref=r3`.

## Asset status — 8 large brand images still to add

Present (28): all 15 speaker photos, all 8 gallery photos, and 5 brand images
(`logo-transparent.png`, `rebbe-cover.png`, `globe.jpg`, `partner-hisachdus.jpg`,
`partner-vaadas-hachinuch.jpg`).

**Missing (8)** — they exceeded the ~256 KB per-file cap of the design-import tool and must be
copied from the design project's `assets/brand/` folder:
`logo-main.png`, `rebbe-portrait-teal.png`, `watercolor-tree-1.png`, `watercolor-tree-2.png`,
`grass-strip.png`, `parchment-farbrengen.png`, `audience-crowd.jpg`, `hatneim-illustration.png`.
The site hides missing images automatically, so it already looks intentional without them —
drop the files in (no code change) and they appear. These are the 1–2 MB source PNGs the brief
says to **compress to WebP/AVIF (with PNG fallback) before shipping**, so add + optimize together.
(`banquet-hall.jpg` from the source package is intentionally unused by this design.)

## Deploying

Upload the folder as-is to any static host (Netlify, Vercel, GitHub Pages, Cloudflare Pages, or
the organizer's server) — no server code needed. Two launch tasks:
1. Set the `og:image` in `index.html` to an **absolute** URL (WhatsApp needs it for the share
   preview) once the domain is known.
2. Compress the 8 large brand PNGs (above) before shipping for a fast mobile load.

Fonts: **Secular One** (display / time numerals) + **Heebo** (body), from Google Fonts.

## Repo

`origin` → https://github.com/sagi770/mekushar-bakol (private). Default branch `main`.
