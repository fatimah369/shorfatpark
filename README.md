# Shorofat Park — Official Prize Draw

A single static page (`index.html`) that runs a live, formal prize draw for
an event: entrants come in from a Google Form / Sheet, a projector-facing
stage shows a spinning wheel, and the draw produces a printable record.

No backend, no build step, no dependencies. Everything runs in the browser.
The only network call is the periodic read of a published Google Sheet CSV —
disconnect the wifi any time after you press **Lock list** and the draw
still works, because the entrant list is already sitting in memory.

## 1. Set up the Google Form and the display-only sheet

1. Create a Google Form that collects whatever you need for contact/eligibility
   (name, phone, etc.) plus the **one field you want shown on the wheel**
   (usually just their name).
2. Open the linked Google Sheet ("Form Responses 1"). This sheet contains
   phone numbers and other private data — **never publish this tab.**
3. Add a **new tab** in the same spreadsheet, containing only the display
   column. In cell A1 of the new tab, use:

   ```
   =QUERY('Form Responses 1'!B:B, "select B where B is not null")
   ```

   (adjust `B:B` to whichever column holds the name/display field). This tab
   now mirrors just that one column, live, with no phone numbers in it.

4. Rename this tab something like `Display`.

## 2. Publish that tab as CSV

1. File → Share → **Publish to web**.
2. In the first dropdown, choose the **`Display`** tab specifically —
   not "Entire document".
3. In the second dropdown, choose **Comma-separated values (.csv)**.
4. Click **Publish** and confirm.
5. Copy the link it gives you. It looks like:

   ```
   https://docs.google.com/spreadsheets/d/e/2PACX-.../pub?gid=123456&single=true&output=csv
   ```

6. Paste that link into the site's **Link Google Sheet** tab and press
   **Connect**.

**Why a separate tab matters:** the published link is public to anyone who
has it — there's no login wall. Publishing only the `Display` tab means the
public link can never expose a phone number, even if someone finds the URL.
Never publish the raw "Form Responses" tab.

If you paste the normal sheet URL (the one in your browser's address bar
when editing) instead of the published link, the site will detect this and
tell you specifically what's wrong — that link isn't fetchable the way a
published CSV is.

## 3. Generate a QR code for the form

Use any QR generator (e.g. the built-in one in Google Forms — click **Send**
→ the link icon → there's a "Shorten URL" option, then use a free QR tool
like `qr-code-generator.com` on that link, or Google Chrome's built-in
"Create QR Code for this page" from the address bar's share icon). Print it
on signage at the event so guests can scan → fill the form → their name
lands in the sheet automatically.

## 4. Event-day sequence

1. **Connect** — Open `index.html`, go to the *Link Google Sheet* tab, paste
   the published CSV URL, press Connect. The list will start refreshing
   every 10 seconds; a green dot means it's live.
   - If wifi is unreliable, you can always fall back to the **Paste** tab
     and paste names in manually — the draw works identically either way.
2. **Verify count** — Watch the entrant count climb as people scan the QR
   code. Check the duplicate counter — if people submitted twice, toggle
   "Remove duplicates" on (it's on by default).
3. **Lock list** — Once registration closes, press **Lock list**. This
   freezes the entrant list, stops polling, and stamps the lock time. From
   this point the list cannot change, network or no network.
4. **Publish the seal** — The **List seal** shown on the setup screen is a
   SHA-256 hash of the entrant list. Screenshot it or read it aloud/post it
   *before* the draw. Since editing even one character of the list produces
   a completely different seal, publishing it beforehand is proof to
   everyone that the list wasn't touched after the fact.
5. **Open stage** — Press **Open draw screen**, then **Fullscreen** on the
   stage. This is what the projector shows.
6. **Spin twice** — Press **Spin the wheel** (or hit `Space`) for the first
   prize, then again for the second. The first winner is automatically
   removed from the pool before the second spin, so nobody can win twice.
7. **Print record** — After both prizes are drawn, press **Print record**.
   This produces a clean one-page printout with both winners, the total
   entrant count, the seal, and the lock timestamp — your paper trail.

`Esc` exits the stage screen back to setup at any time.

## 5. About the wheel and the 24-segment cap

The wheel is a **presentation device only** — it never decides the winner.

On every spin, the actual winner is chosen first, instantly, using
`crypto.getRandomValues` with rejection sampling (so there's no bias toward
any name) over the **full** entrant list — even if that list has 5, 50, or
5,000 names.

A wheel with hundreds of names on it is unreadable from across a room, so
the site caps what's drawn on screen at **24 segments**. If the list is
longer than 24:

1. The real winner is picked from the full list first.
2. 23 other names are picked at random to fill out the wheel.
3. Those 24 names (winner included) are shuffled and drawn on the wheel,
   which then spins and lands on the winner's segment.

The record strip at the bottom of the stage always reads the true entrant
count (e.g. "500 entries"), never 24 — the cap only affects what's rendered
on the wheel, not what the draw is based on.

## 6. Redeploying after edits

This is a static site with one HTML file — there's no build step.

- **Netlify Drop**: after editing `index.html`, go to
  [app.netlify.com/drop](https://app.netlify.com/drop) and drag the folder
  in again. If you want it to land on the *same* URL as before, drag it
  into your existing site's **Deploys** tab in the Netlify dashboard instead
  of the drop page (Deploys → drag and drop the folder onto the deploy list).
- **Cloudflare Pages / Netlify via Git**: if the project is instead connected
  to a GitHub repository, just commit and push your change to the connected
  branch — the host rebuilds and redeploys automatically within a minute or
  two, no build command needed (leave the build command blank; the output
  directory is the repo root).

## Files

- `index.html` — the entire site (admin screen + stage + wheel). Drag this
  file (or the folder containing it) to your static host.
- `README.md` — this file.
