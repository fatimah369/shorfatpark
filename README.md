# Shorofat Park — Official Prize Draw

Two static pages that run a live, formal prize draw for an event: entrants
come in from a Google Form / Sheet, a projector-facing stage shows a
spinning wheel, and the draw produces a printable record.

No backend, no build step, no dependencies. Everything runs in the browser.

## Two pages — one private, one public

- **`admin.html`** — private. This is where *you* connect the Google Sheet,
  review the entrant count, set prizes, lock the list, and see the seal.
  **Never publish or share this link.** Bookmark it for yourself only.
- **`index.html`** — public. This shows nothing but the wheel. It's safe to
  put on a projector, share, or leave as the site's default URL — it does
  nothing until your admin page sends it data, and has no controls to fetch
  anything itself.

**How they connect:** on `admin.html`, the "Open draw screen" button is a
real link that opens `index.html` in a new browser tab. Once that tab
loads, it quietly announces itself back to the admin tab, which replies
with the entrant list, prizes, and seal — all in memory, via the browser's
own `postMessage`, never through localStorage or a server. Keep both tabs
open on the same laptop during the event: you drive the draw from the admin
tab, the stage tab is what's on the projector.

If someone opens the public URL directly (bookmarked it, found the link)
without your admin tab open, they just see an idle "waiting for connection"
screen — no entrant data, no way to configure anything.

The only network call anywhere is the periodic read of a published Google
Sheet CSV, made by `admin.html`. Disconnect the wifi any time after you
press **Lock list** and the draw still works, because the entrant list is
already sitting in the stage tab's memory.

## 1. Set up the Google Form and the display-only sheet

1. Create a Google Form that collects whatever you need for contact/eligibility
   (name, phone, etc.) plus the **one field you want shown on the wheel**
   (usually just their name).
2. Open the linked Google Sheet ("Form Responses 1"). This sheet contains
   phone numbers and other private data — **never publish this tab.**
3. Add a **new tab** in the same spreadsheet, containing only the display
   column. In cell A1 of the new tab, use:

   ```
   =QUERY('Form Responses 1'!J:J, "select J where J is not null", 1)
   ```

   (`J` here is an example — use whichever column letter actually holds the
   name/display field in your Form Responses sheet; check the header row to
   find it. The trailing `, 1` tells QUERY the source has one header row,
   so it isn't miscounted as data.) This tab now mirrors just that one
   column, live, with no phone numbers or emails in it.

4. Rename this tab something like `Display`.

## 2. Publish that tab as CSV

1. On the **`Display`** tab (make sure it's the active tab), go to
   File → Share → **Publish to web**.
2. In the first dropdown, choose the **`Display`** tab specifically —
   not "Entire document", and not the raw Form Responses tab.
3. In the second dropdown, choose **Comma-separated values (.csv)**.
4. Click **Publish** and confirm.
5. Copy the link it gives you. It looks like:

   ```
   https://docs.google.com/spreadsheets/d/e/2PACX-.../pub?gid=0&single=true&output=csv
   ```

6. On `admin.html`, paste that link into the **Link Google Sheet** tab and
   press **Connect**.

**Why a separate tab matters:** the published link is public to anyone who
has it — there's no login wall. Publishing only the `Display` tab means the
public link can never expose a phone number or email, even if someone finds
the URL. Never publish the raw "Form Responses" tab, and never publish the
whole document — always publish the specific `Display` tab from the first
dropdown.

If you accidentally publish the wrong thing, go back to File → Share →
Publish to web, find it in the list, and **Stop publishing** it immediately.

If you paste the normal sheet URL (the one in your browser's address bar
when editing) instead of the published link, or a link published as "Web
page" instead of CSV, the site will detect this and tell you specifically
what's wrong.

## 3. Generate a QR code for the form

Use any QR generator (e.g. the built-in one in Google Forms — click **Send**
→ the link icon → there's a "Shorten URL" option, then use a free QR tool
like `qr-code-generator.com` on that link, or Google Chrome's built-in
"Create QR Code for this page" from the address bar's share icon). Print it
on signage at the event so guests can scan → fill the form → their name
lands in the sheet automatically.

## 4. Event-day sequence

1. **Open `admin.html`** (your private link — not the public one) and
   connect the sheet as above. A green dot means it's live, refreshing
   every 10 seconds.
   - If wifi is unreliable, use the **Paste** tab instead and paste names
     in manually — the draw works identically either way.
2. **Verify count** — Watch the entrant count climb as people scan the QR
   code. Check the duplicate counter — if people submitted twice, toggle
   "Remove duplicates" on (it's on by default).
3. **Lock list** — Once registration closes, press **Lock list**. This
   freezes the entrant list, stops polling, and stamps the lock time.
4. **Publish the seal** — The **List seal** shown on the admin page is a
   SHA-256 hash of the entrant list. Screenshot it or read it aloud/post it
   *before* the draw. Since editing even one character of the list produces
   a completely different seal, publishing it beforehand is proof to
   everyone that the list wasn't touched after the fact.
5. **Open draw screen** — Click the button; it opens the public stage
   (`index.html`) in a new tab. Move that tab to the projector display and
   press **Fullscreen**. Keep the admin tab open in the background on your
   own screen — it's how you drive the draw.
6. **Spin twice** — On the stage tab, press **Spin the wheel** (or hit
   `Space`) for the first prize, then again for the second. The first
   winner is automatically removed from the pool before the second spin.
7. **Print record** — After both prizes are drawn, press **Print record**
   on the stage tab. This produces a clean one-page printout with both
   winners, the total entrant count, the seal, and the lock timestamp.

## 5. About the wheel and the 24-segment cap

The wheel is a **presentation device only** — it never decides the winner.

On every spin, the actual winner is chosen first, instantly, using
`crypto.getRandomValues` with rejection sampling (so there's no bias toward
any name) over the **full** entrant list — even if that list has 5, 50, or
10,000 names (tested).

A wheel with hundreds of names on it is unreadable from across a room, so
the site caps what's drawn on screen at **24 segments**. If the list is
longer than 24:

1. The real winner is picked from the full list first.
2. 23 other names are picked at random to fill out the wheel.
3. Those 24 names (winner included) are shuffled and drawn on the wheel,
   which then spins and lands on the winner's segment.

The record strip at the bottom of the stage always reads the true entrant
count (e.g. "10000 entries"), never 24 — the cap only affects what's
rendered on the wheel, not what the draw is based on.

## 6. Redeploying after edits

Static files, no build step — **always deploy both `admin.html` and
`index.html` together**, at the same site.

- **Netlify Drop**: after editing, go to
  [app.netlify.com/drop](https://app.netlify.com/drop) and drag the whole
  folder in (both files). If you want it to land on the *same* URL as
  before, drag it into your existing site's **Deploys** tab in the Netlify
  dashboard instead of the drop page.
- **Cloudflare Pages / Netlify via Git**: if the project is connected to a
  GitHub repository, just commit and push — the host redeploys
  automatically within a minute or two, no build command needed (leave the
  build command blank; the output directory is the repo root).

After redeploying, your private admin link stays `<your-site>/admin.html`
and the public stage link stays `<your-site>/` (or `/index.html`) — nothing
changes about those URLs across redeploys.

## Files

- `index.html` — the public stage (wheel only). This is what goes on the
  projector and is safe to be the site's public URL.
- `admin.html` — the private setup page (entrant list, Google Sheet, prizes,
  seal, lock). **Do not share this link.**
- `README.md` — this file.
