# מקושר בכל 15 — אתר הכנס

Single-page, mobile-first, Hebrew-RTL landing site for **מקושר בכל 15** — the 15th annual
Lubavitch family conference by התאחדות החסידים ליובאוויטש.
Tuesday, ז׳ מנחם־אב · 21.7.2026 · מרכז הכנסים הבינלאומי, אשקלון.

This is the production implementation of the approved Claude Design prototype
(`מקושר בכל 15.dc.html`). It ships as **plain static files — no build step, no framework,
no runtime dependency**. All content is rendered dynamically at load time from a single
JSON file.

---

## Structure

```
mekusar-website/
├── index.html          ← the whole site: markup skeleton + inline CSS + vanilla-JS renderer
├── data/
│   └── event.json      ← SINGLE SOURCE OF TRUTH — every string, time, name and quote
└── assets/
    ├── brand/          ← logo, Rebbe photos, watercolor trees, grass strip, illustrations
    ├── speakers/       ← speaker headshots (filename = speaker id in event.json)
    └── gallery/        ← photos from previous conferences
```

`index.html` fetches `data/event.json` and builds the page from it. **Changing a session
title, a time, a speaker bio or a FAQ answer in `event.json` changes the page** — treat it
as the CMS. There is no hardcoded Hebrew content in `index.html` except UI microcopy
(countdown labels, "שים לב", accordion labels, etc.).

## Asset status — 8 large brand images still to add

All 15 speaker photos, all 8 gallery photos, and 5 brand images (`logo-transparent.png`,
`rebbe-cover.png`, `globe.jpg`, `partner-hisachdus.jpg`, `partner-vaadas-hachinuch.jpg`) were
imported and are present under `assets/`.

The following **8 large brand images could not be auto-imported** (each exceeds ~256 KB, the
per-file limit of the design-import tool) and need to be copied in from the design project's
`assets/brand/` folder:

```
assets/brand/logo-main.png              (hero logo)
assets/brand/rebbe-portrait-teal.png    (Rebbe-quotes section)
assets/brand/watercolor-tree-1.png      (hero garnish)
assets/brand/watercolor-tree-2.png      (hero + testimonials garnish)
assets/brand/grass-strip.png            (footer top edge)
assets/brand/parchment-farbrengen.png   (banquet corner texture)
assets/brand/audience-crowd.jpg         (13:00 keynote card background)
assets/brand/hatneim-illustration.png   (banquet illustration)
```

Good news: the site is built to **degrade gracefully** — any image that isn't present is
hidden automatically (no broken-image icons), so the page already looks intentional without
them. Drop the 8 files in and they light up with no code change.

These are exactly the 1–2 MB source PNGs the brief (§7.7) says to **compress to WebP/AVIF
(with a PNG fallback) before shipping** — so treat "add them" and "optimize them" as one step.
`banquet-hall.jpg` from the original package is intentionally not used by this design.

Relationships inside `event.json`:
`schedule[].sessions[].speaker_id → speakers[].id`,
`… .hall_id → halls[].id`, `… .track_id → tracks[].id`,
`speakers[].photo` / `gallery.images[]` → paths under `assets/`.

---

## Running locally

The site loads its content with `fetch('data/event.json')`, and browsers block `fetch` over
the `file://` protocol. So **serve the folder over HTTP** — don't just double-click
`index.html` (you'll get a friendly "run a local server" message if you do).

From this directory, any one of:

```bash
npx serve            # Node
# or
python -m http.server 8000
# or
php -S localhost:8000
```

Then open the URL it prints (e.g. `http://localhost:3000` / `http://localhost:8000`).

---

## Deploying

Upload the folder as-is to any static host (Netlify, Vercel, GitHub Pages, Cloudflare Pages,
or the organizer's own web server). No server-side code is required.

Two things to do for a public launch:

1. **Open Graph image URL (WhatsApp previews).** This link spreads on WhatsApp, so the share
   preview matters. In `index.html`, the `og:image` is a relative path. WhatsApp/Facebook
   require an **absolute** URL — replace it with your live domain, e.g.
   `https://your-domain.co.il/assets/brand/logo-main.png`. (A dedicated 1200×630 banner reads
   even better than the logo if you have one.)

2. **Compress the large brand PNGs.** `rebbe-portrait-teal.png`, the watercolor trees and
   `hatneim-illustration.png` are heavy source PNGs. Convert them to WebP/AVIF (with a PNG
   fallback) before shipping for a faster mobile load. Speaker photos are already small
   avatars — leave them as-is.

---

## What's implemented

Everything from the design brief:

- Sticky header, hero with logo + big "15", date pills, live **countdown** to
  2026-07-21 10:00 Asia/Jerusalem (auto-switches to "הכנס בעיצומו!" during the event and a
  thank-you note after), primary CTA to Tickchak, and an **Add-to-calendar (ICS)** download.
- The Rebbe + Hayom-Yom "identity moment", About ("תחנת כוח להטענה"), the 9 tracks strip.
- The full **schedule timeline** with oversized alternating time numerals, parallel-session
  card grids, the 13:00 keynote hero card with pull-quote, break/meal dividers, the pre-10:00
  gates/shacharit rows, and Ronen Peled's closed-workshop callout.
- Banquet (17:00) + Farbrengen (19:00) section, full speakers grid (initials avatars for
  those without a photo), swipeable testimonials, Rebbe quotes, gallery, FAQ accordion,
  getting-there (Waze / Google Maps / tap-to-call hotline), registration + perks, sponsors,
  and the footer with the yechi line, dedication and the required build credit.
- Sticky mobile bottom CTA after the hero.
- RTL + logical CSS properties, visible focus states, lazy-loaded below-the-fold images,
  and `prefers-reduced-motion` honored (all entrance/scroll animations disable).

Fonts: **Secular One** (display / time numerals) + **Heebo** (body), loaded from Google Fonts.

---

## Editing content

Open `data/event.json`. It is the only place to change copy. A few notes:

- Religious phrasing (honorifics, the yechi line, ב״ה, the dedication) is intentionally
  verbatim from the printed program — keep it exact.
- A speaker with `"photo": null` renders as a teal initials avatar automatically
  (e.g. הרב עופר מיודובניק, צוות התנעים).
- The footer credit (`site.footer_credit`) is required by the site owner and links to
  x-technologies.co.il.
- Ticket prices are deliberately not on the site — Tickchak owns them; every CTA links there.

---

_Design → implementation of `מקושר בכל 15.dc.html`. Content extracted from the printed
program (תכניה)._
