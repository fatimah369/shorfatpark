# Shorofat Park — Official Prize Draw

One static page (`index.html`) that runs a live prize draw: paste in
entrant names (or drop a CSV), a projector-facing spinning wheel picks two
winners.

No backend, no build step, no dependencies, no network calls at all.
Everything runs in the browser, entirely in memory — reloading the page
is always a clean slate.

## How it works

One page, two modes:

- **Setup** (what you see first): paste names one per line, or drop a
  CSV file (or click to choose one) — either way they land in the same
  list. Set the two prize names. A live count shows total entrants and
  how many duplicates were found.
- **Stage**: press **Start draw**. This hides setup, goes fullscreen, and
  shows nothing but the event title, the wheel, and the spin button —
  what the projector displays. Press `Esc` to leave fullscreen and return
  to setup at any time.

## Getting the entrant list in

**Paste**: copy names from anywhere (a spreadsheet, a form export, a plain
list) and paste directly into the textarea, one name per line.

**CSV file**: drag a `.csv` file onto the dropzone under the textarea, or
click it to choose a file. It reads the file in your browser (nothing is
uploaded anywhere), takes the **first column** of each row, and
automatically skips the first row if it looks like a header (e.g. "Name",
"الاسم"). The names get appended into the same textarea, so you can review
or edit them before starting.

If your source data has other columns (phone numbers, emails, timestamps,
etc.), that's fine — only the first column is used, so as long as the name
is in column A of your CSV, nothing else comes through.

## Event-day sequence

1. Paste your list or drop the CSV.
2. Check the count — the duplicate counter tells you if any names repeat;
   toggle "Remove duplicates" off if you want to keep them (on by
   default).
3. Fill in both prize names.
4. Press **Start draw**. This goes fullscreen automatically.
5. Press **Spin the wheel** (or hit `Space`) for the first prize, then
   again for the second. The first winner is automatically removed from
   the pool before the second spin, so nobody can win twice.
6. Once both prizes are drawn, a results list appears with both winners
   and the true entrant count, plus a **Print** button for a clean
   one-page record.
7. Press `Esc` to leave the stage and return to setup.

## About the wheel and the 24-segment cap

The wheel is a **presentation device only** — it never decides the winner.

On every spin, the actual winner is chosen first, instantly, using
`crypto.getRandomValues` with rejection sampling (so there's no bias
toward any name) over the **full** entrant list, however long it is.

A wheel with hundreds of names on it is unreadable from across a room, so
the site caps what's drawn on screen at **24 segments**. If the list is
longer than 24:

1. The real winner is picked from the full list first.
2. 23 other names are picked at random to fill out the wheel.
3. Those 24 names (winner included) are shuffled and drawn on the wheel,
   which then spins and lands on the winner's segment.

The results screen always shows the true entrant count, never 24 — the
cap only affects what's rendered on the wheel, not what the draw is based
on.

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
- **Cloudflare Pages / Netlify via Git**: if connected to
  `github.com/fatimah369/shorfatpark`, just commit and push — the host
  redeploys automatically (leave the build command blank, output
  directory is the repo root).

## Files

- `index.html` — the entire site.
- `README.md` — this file.
