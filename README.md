# 🎃 Spooky AI Photo Booth (Halloween edition)

A PWA photo booth: guests unlock with a **PIN bought at the counter**, pick a **character** (no rush), take **one shot** with a **10-second countdown**, and OpenAI `gpt-image-1-mini` makes two pictures **while preserving their faces**: **frame window 1** is a photoreal shot of them as the chosen character, and **frame window 2** is that same picture re-rendered as a **3D animated-movie still**. Both fill your uploaded frame, then guests **scan a QR to download** or **print 5R (5×7")**.

```
/                      ← GitHub Pages site (frontend)
├─ index.html          booth app
├─ download.html       page opened by the QR code
├─ app.js · styles.css · config.js
├─ manifest.webmanifest · sw.js     (PWA)
├─ frames/             your 2 layouts (windows cut transparent) + thumbnails
├─ icons/
└─ backend/            Google Apps Script (copy into script.google.com)
   ├─ Code.gs
   └─ appsscript.json
```

## Guest flow
1. **Animated intro** (rising moon, bat swarm, dripping title, lightning, fog) → *Tap to Begin*. First visit shows the **Install app** banner.
2. **PIN** — on-screen keypad (works with a keyboard too). Re-entering a PIN resumes the session.
3. **Choose character** — no timer; tap one, then *Cast the Spell*.
4. **Camera** — one shot only. Tapping *I'm Ready* starts a **10-second countdown** (the only timer in the app). Fallback: upload a photo if the camera is blocked.
5. **Brewing (2 steps)** — Step 1 makes the character photo; Step 2 turns that exact picture into a 3D animated version (the guest sees step 1 while step 2 runs). Each step is its own server call, so neither hits Apps Script time limits. If one fails, *Try Again* repeats only that step — still one shot.
6. **Pick your frame** — both of your layouts shown **with the guest's real photos already in them**
7. **Result** — QR download, paid prints, save to device, *Done — Next Guest* (auto-resets after 3 min).
   - **Downloads (free):** the QR page and *Save 3 Photos* give the guest **3 pictures** — the photo collage, the character photo, and the 3D animated photo.
   - **Prints (paid):** *Print 5R Photos* lets the guest pick which of the 3 to print and how many copies (max 10 each). The total is **PRINT_PRICE × prints** (default ₱50 each). The guest pays at the counter, staff type the **STAFF_CODE** on the booth, and the print dialog opens with one 5×7" page per copy. Every order is logged in the **Prints** sheet (order ID, PIN, copies of each photo, amount). Wrong codes are limited to 5 tries per 10 minutes.

**Characters:** Vampire Royalty, Witch & Warlock, Spellcaster, Moon Wolf, Calavera, Zombie Prom, Pumpkin Royalty, Ghost Pirate, Mummy Explorer. Every costume prompt forbids masks, face paint, fangs and prosthetics so faces stay untouched, and your **Identity Prompt is sent first in every request**.

---

## Setup (about 15 minutes)

### 1. Backend — Google Apps Script
1. Create a new **Google Sheet** (e.g. "Photo Booth PINs").
2. **Extensions → Apps Script.** Paste `backend/Code.gs` into `Code.gs`.
   In **Project Settings** tick *Show "appsscript.json"* and paste `backend/appsscript.json`.
3. Run **`setup`** once (authorize when asked). It creates the `PINs` sheet, a Drive folder **"AI Photo Booth – Photos"**, and a daily cleanup.
4. **Project Settings → Script properties → Add** `OPENAI_API_KEY` = `sk-...` (the only thing kept outside the sheet, so the key never shows in a spreadsheet).
5. Open the new **Settings** tab and fill in **STAFF_CODE** (e.g. `4821`) — needed for paid prints. Check the prices and texts while you're there.
6. **Deploy → New deployment → Web app** · Execute as **Me** · Who has access **Anyone** → copy the URL ending in `/exec`.
7. Reload the Sheet → menu **🎃 Photo Booth → Generate PINs…** to print PINs for the counter. Also: *Test OpenAI key*, *Reset selected PIN* (lets a guest re-shoot), *Void PIN*, *Today's stats* (includes prints and print revenue).

> After editing `Code.gs` later: **Deploy → Manage deployments → Edit → New version** (keeps the same URL).

### 2. Frontend — GitHub Pages
1. Open `config.js` and paste your `/exec` URL into `API_URL`.
2. Create a GitHub repo, upload **everything in this folder** (keep `.nojekyll`).
3. **Settings → Pages → Deploy from branch → `main` / root.** Your booth is at `https://<you>.github.io/<repo>/`.
4. After any update, bump `VERSION` in `sw.js` so installed devices refresh.

**Demo PIN:** enter **000000** (the `DEMO_PIN` setting) to run the full flow with **real AI**, which uses OpenAI credit. Each use opens its own `DEMO-…` row in the PINs tab and is never charged to a real PIN. Demo prints use the real STAFF_CODE and are logged in the Prints tab at ₱0. `DEMO_DAILY_LIMIT` (default 20 per day) stops a leaked demo PIN from running up your OpenAI bill, and today's stats count demo sessions separately. The demo PIN is never sent to guests' browsers. If `API_URL` is not set yet (offline preview), 000000 shows plain stand-in pictures instead.

**PIN Slip price:** guests see “Buy your PIN Slip at the Booth Counter for ₱150.00” on the opening and PIN screens. Change `PIN_PRICE` in `config.js`.

### 3. Printing 5R
*Print 5R* builds a 1500×2100 px (5×7" @ 300 dpi) page and opens the print dialog with `@page size: 5in 7in`, margins 0. In the dialog choose your photo printer, **5×7 / 127×178 mm paper**, scale **100% / Actual size**, borderless if offered. The frame is 9:16, so by default it's centred over a blurred fill of itself; set `PRINT_FIT: 'cover'` in `config.js` to fill the page instead (crops top/bottom). For a kiosk with no dialog, launch Chrome with `--kiosk-printing`.

---

## Customising from the Google Sheet (no redeploy)
`setup` creates two tabs. Edit them any time — the booth picks up changes within a minute, or straight away with **🎃 Photo Booth → Apply changes now**. That menu item also double-checks key values (and warns if STAFF_CODE is empty).

**Settings tab** (Section · Setting · Value · What it does)

| Section | Settings |
|---|---|
| Guest screens | `BOOTH_NAME`, `THEME_TITLE` (big animated title), `SUBTITLE`, `TAGLINE` |
| PIN Slip | `PIN_PRICE` (150), `CURRENCY` (₱), `DEMO_PIN` (000000, real AI — leave empty to disable), `DEMO_DAILY_LIMIT` (20) |
| Prints | `PRINTING_ENABLED` (checkbox), `PRINT_PRICE` (50), `MAX_COPIES` (10), `STAFF_CODE` |
| Booth timing | `CAMERA_COUNTDOWN` (10 s), `RESULT_IDLE_SECONDS` (180), `MIRROR_OUTPUT` |
| Layouts | `LAYOUTS` — `frame1, frame2` (remove or reorder to change what guests see) |
| AI | `AI_ENGINE` (direct / background), `OPENAI_MODEL`, `AI_TEXT_MODEL`, `IMAGE_QUALITY`, `INPUT_FIDELITY` (auto), `MAX_ATTEMPTS`, `IDENTITY_PROMPT`, `ANIMATED_STYLE` |
| Privacy | `RETENTION_DAYS` (7), `PUBLIC_PHOTO_LINKS` (on) |

`STAFF_CODE`, the prompts and the AI options stay on the server — guests' browsers never receive them. Keep the Sheet itself private.

**Characters tab** — one row per character: *Enabled* (checkbox) · ID · Name · Emoji · Tagline · Color · Costume · Scene. Untick to hide, edit the costume/scene text to change what the AI makes, add a row for a new character. Row order = order on the booth.

If a newer version of `Code.gs` adds settings, run **Add missing settings / characters** — it adds the new rows and never overwrites yours.

**New frames:** make a 940×1672 PNG with transparent photo windows, add it to `LAYOUTS` in `config.js` (with each window's centre, size and angle), then list its id in the `LAYOUTS` setting.

## Photos tab (every saved picture, with lh3 links)
Each picture the booth saves to Drive gets a row: **Time · PIN · Type** (guest photo / AI character photo / AI 3D animated photo / collage / test) **· Character · Layout · Preview · lh3 URL · Drive link · File name · File ID · Status**.
- **lh3 URL** = `https://lh3.googleusercontent.com/d/<FILE_ID>`, a direct image link you can use in other apps and pages.
- For lh3 links and the **Preview** thumbnails to work, the `PUBLIC_PHOTO_LINKS` setting (on by default) shares each photo as "anyone with the link can view". The IDs are long and random, but anyone who has a link can see that photo. Untick the setting to keep photos private. If your Google Workspace admin blocks link sharing, Status says so.
- When auto-delete removes a session's photos, their rows change to *Deleted after 7 days (in Drive trash for 30 days)*.
- **📷 Check photo storage** shows the folder link, how many of each type are in it (and in the trash), the newest file, auto-delete status and the booth's last request. It also adds any Drive photos missing from the tab. **📷 Refresh Photos tab** re-checks each row (in Drive / in trash / missing).

## If the AI is not working
1. Sheet menu **🎃 Photo Booth → 🧪 Test AI generation**. It runs one real picture from the newest guest photo and shows the engine, model, each status with timings, and **OpenAI's exact error**. If there's no guest photo yet, run the demo PIN once on the booth first.
2. The booth itself shows the same error in small text under the guest message ("Details for staff: …"), and the PINs tab saves it in the **Note** column.

| What the error says | Fix |
|---|---|
| `Incorrect API key` / 401 | Re-paste `OPENAI_API_KEY` in Script properties (no spaces) |
| `organization must be verified` | platform.openai.com → Settings → Organization → Verify |
| `quota` / `billing` / 429 | Add credit at platform.openai.com → Billing |
| `model … does not exist` | Check `OPENAI_MODEL` / `AI_TEXT_MODEL` in Settings |
| `Timeout` / `Exceeded maximum execution time` | Set `AI_ENGINE` = `background` |
| `safety system` | Guest taps **Pick Another Character** (same photo) |

**Engines.** `direct` (default) makes one `/images/edits` call that waits for the picture. The photo is sent as JSON `images:[{image_url: data URL}]`, the same method proven in production booths, and the prompt reaches the image model word for word. Apps Script lets a single request run for minutes. `background` is a backup: it starts an OpenAI job and the booth checks every 3 s. A job stuck in the queue for more than 75 s is re-sent once, and a step stops after 5 minutes. It adds a small helper-model cost.

**Check the deployment:** open your `/exec` URL in a browser. It should say `AI Photo Booth API v12 is running…`. A lower number, or no version, means Apps Script is still serving an old deployment: Deploy → Manage deployments → ✏️ Edit → Version: New version → Deploy.

**Model retirement:** OpenAI retires `gpt-image-1-mini` on **1 Dec 2026**. Change `OPENAI_MODEL` in Settings before then (e.g. `gpt-image-2`) and run the Test AI menu item.

## Notes & limits
- Two AI images per guest = 2 OpenAI image jobs, one after the other (character → animated). The background engine also adds a small charge for the `AI_TEXT_MODEL` helper. Cost and speed depend on the `IMAGE_QUALITY` setting.
- Apps Script handles ~30 simultaneous requests — plenty for a booth.
- Photos are private in your Drive; the QR link uses a random share code, not the Drive file ID, and expires after `RETENTION_DAYS`.
- Camera requires HTTPS (GitHub Pages provides it).
