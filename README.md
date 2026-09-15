# Shorofat Park — Official Prize Draw

One static page (`index.html`) that runs a live prize draw: entrant names
come in live from a published Google Sheet (or paste/CSV as a fallback),
and a projector-facing name reel picks two winners.

No backend, no build step, no dependencies. Everything except the Google
Sheet fetch runs entirely in the browser, in memory — reloading the page
is always a clean slate. Once entrants are loaded, the draw itself needs
no network at all.

## How it works

One page, two modes:

- **Setup** (what you see first): connects automatically to the published
  Google Sheet on load — no click needed. Set the two prize names. A live
  count shows total entrants and how many duplicates were found.
- **Stage**: press **Start draw**. This hides setup, goes fullscreen, and
  shows nothing but the event title, the name reel, and the draw button —
  what the projector displays. Press `Esc` to leave fullscreen and return
  to setup at any time.

## Getting the entrant list in

**Google Sheet (default, live)**: the site is pre-connected to a specific
published CSV link (see below), refreshing every 10 seconds. A dot next
to the status line shows green when live. This is a **display-only**
sheet tab — it contains names only, never phone numbers or emails — see
"About the connected sheet" below for why that separation matters.

**Paste** (fallback): click the "لصق يدوي" / "Paste" tab to switch off the
live sheet and type or paste names directly, one per line.

**CSV file** (fallback, under the Paste tab): drag a `.csv` file onto the
dropzone, or click it to choose a file. It reads the file in your browser
(nothing is uploaded anywhere), takes the **first column** of each row,
and automatically skips the first row if it looks like a header (e.g.
"Name", "الاسم"). Useful if the wifi drops — download the sheet as CSV
and drop it in instead.

If your source data has other columns (phone numbers, emails, timestamps,
etc.), that's fine in the CSV file case — only the first column is used.

## About the connected sheet

The site is pointed at a tab in the Google Sheet that was created
specifically to be safe to publish: it contains **only the name column**,
built from the real form-responses sheet with a formula like:

```
=QUERY('Form Responses 1'!J:J, "select J where J is not null", 1)
```

That tab, and *only* that tab, is published to the web as CSV (File →
Share → Publish to web → pick that tab → Comma-separated values). The raw
"Form Responses" sheet — which has phone numbers and emails — is never
published. If you ever need to point this at a different sheet, always
publish a display-only tab, never the raw responses sheet, and never
"Entire document".

To change which sheet it connects to: open `index.html`, find the
`sheetUrl` input's `value` attribute in the setup section, and replace it
with the new published CSV link (ending in `output=csv`).

## Event-day sequence

1. Open the page — it connects to the sheet automatically. Watch the
   count climb as people register.
2. Check the duplicate counter — toggle "Remove duplicates" off if you
   want to keep them (on by default).
3. Fill in both prize names.
4. Press **Start draw**. This goes fullscreen automatically and stops the
   live sheet refresh — the list is now fixed for the draw.
5. Press **Spin the wheel** (or hit `Space`) for the first prize, then
   again for the second. The first winner is automatically removed from
   the pool before the second spin, so nobody can win twice.
6. Once both prizes are drawn, a results list appears with both winners,
   the true entrant count, and the date/time, plus a **Print** button for
   a clean one-page record.
7. Press `Esc` to leave the stage and return to setup.

## About the name reel and the 40-row cap

The reel is a **presentation device only** — it never decides the winner.

On every draw, the actual winner is chosen first, instantly, using
`crypto.getRandomValues` with rejection sampling (so there's no bias
toward any name) over the **full** entrant list, however long it is.

Scrolling hundreds of names past is unreadable, so the reel strip is built
from about 40 rows. If the list is longer than 40:

1. The real winner is picked from the full list first.
2. 39 other names are picked at random to fill out the strip.
3. The strip scrolls and settles with the winner's row — always the last
   one — centred under the gold highlight band.

(If there are fewer than 40 entrants, the pool is repeated to fill the
strip — still decoration; the winner was already chosen from the real
list before the strip was even built.)

The results screen always shows the true entrant count, never 40.

## Design

Saudi National Day 96 ("عزّنا بطبعنا") green-and-gold on the stage screen,
Shorofat Park cream-and-gold on the setup screen — tied together by the
same gold accent and an original line-art rendition of Shorofat Park's
balcony/arch motif, used as the header icon. Tajawal for interface text,
Noto Naskh Arabic for the winner reveal. Full RTL support with an
Arabic/English toggle.

## Redeploying after edits

Static file, no build step.

- **Netlify Drop**: after editing `index.html`, go to
  [app.netlify.com/drop](https://app.netlify.com/drop) and drag it in
  again. To land on the same URL as before, drag it into your existing
  site's **Deploys** tab in the Netlify dashboard instead.
- **Cloudflare Pages / Netlify via Git**: connected to
  `github.com/fatimah369/shorfatpark` — just commit and push, the host
  redeploys automatically (leave the build command blank, output
  directory is the repo root).

## Files

- `index.html` — the entire site.
- `README.md` — this file.
