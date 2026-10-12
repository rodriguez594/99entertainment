# 99ENT: Website Plan

99ENT (99entertainment) is the production arm of 99. It does music videos, promo videos, music production, and mixing and mastering. It sits alongside 99MGMT, 99MS, 99RIM and 99WRLDWIDE.

- **Design reference:** [donprod.uk](https://donprod.uk/) (DON PROD). We use it for structure and motion only.
- **Family reference:** the `99management`, `99ms` and `99rim` repos. We use them for look, stack and deploy flow.

> **Inspired by, not copied.** We take DON PROD's *system*: a cinematic monospace HUD around full-bleed video. We dress it in 99's own identity. No DON PROD assets, copy or code.

**Scope for v1: videos only.** Music and audio work can come later; see §9.

---

## 1. Domains and the publish flow

| Address | What it is | Who sees it |
|---|---|---|
| **99entertainment.dk** | Live site | Everyone. It only changes when someone presses **Push live** |
| **prod.99entertainment.dk** | Prod (review) site. All code changes and newly posted videos land here first | Public URL, but `noindex`, a yellow **PROD** badge, and drafts are visible only when logged in |
| **admin.99entertainment.dk** | Admin panel. Post, edit, reorder and hide videos | Logged-in 99 admins only |

```
 admin.99entertainment.dk           prod.99entertainment.dk                99entertainment.dk
 ┌──────────────────────┐  draft   ┌──────────────────────────┐  Push live  ┌──────────────────┐
 │ upload video, title, │ ───────▶ │ shows live + DRAFT items │ ──────────▶ │ shows live items │
 │ artist, category …   │          │ runs newest code         │   button    │ runs pushed code │
 └──────────────────────┘          └──────────────────────────┘             └──────────────────┘
      code changes (me / Claude) ─────────▲
```

**One button pushes both content and code.**

- **Content:** every video marked *ready* goes from `draft` to `live`.
- **Code:** the live site is updated to the exact commit you just reviewed on prod.

**What the Push live button does** (shown on prod only, and only when logged in as an admin):

1. It opens a summary of what will change: "3 new videos, 1 edited, 4 code changes since last push (list)".
2. On confirm, a server function checks that you are an admin. Then it:
   - publishes the ready videos, and
   - moves the live site to prod's current commit.
3. Live updates in ~30–60s. Each push is logged with who, when, which commit and which videos, so **Undo last push** can roll back to the previous one.

**Hiding a video is instant.** If something has to come down (legal, the artist asks), "Hide" in admin removes it from live immediately. It doesn't wait for a push.

**Unreleased videos stay private.** Drafts are never in the public data. Prod fetches them only with an admin login, and the video files use signed URLs. That way a music video can't leak before its release date.

---

## 2. Hosting: GitHub Pages (family default)

**Rule for 99:** choose GitHub Pages first. Moving to Cloudflare Pages is always an option later; see the comparison below.

This copies the 99RIM setup (prod repo + live repo) and adds an admin repo:

| Repo | Domain | Updates |
|---|---|---|
| `rodriguez594/99entertainment-prod` | prod.99entertainment.dk | Every change lands here first |
| `rodriguez594/99entertainment` (this repo) | 99entertainment.dk (+ `www`) | Only via **Push live** |
| `rodriguez594/99entertainment-admin` | admin.99entertainment.dk | Every change (internal tool) |

- Each repo has its own `CNAME` file. **Never copy `CNAME` between repos.**
- Prod shows the yellow PROD badge and `<meta name="robots" content="noindex">`, as on 99RIM.

### How Push live works

1. The button on prod calls a **Supabase Edge Function** called `push-live`. Supabase already runs 99RIM's AI function. The function checks that the caller is an admin, publishes the *ready* videos, and logs the push.
2. It then starts a GitHub Action in this repo (`sync-from-prod.yml`), passing prod's current commit.
3. The Action:
   - checks out `99entertainment-prod` at that commit,
   - copies everything except `CNAME`, `.git` and `.github`,
   - commits "Push live from prod <sha>",
   - deploys to Pages.

   This repo's Pages source is "GitHub Actions". Live is then exactly what was reviewed on prod, about 1–2 minutes after pressing the button.
4. **Undo** runs the same Action with the previous push's commit.
5. **Secrets:** a fine-grained GitHub token, allowed to run Actions on this repo only, is stored as a Supabase secret. All repos are public, so the Action needs no extra token to read prod.

### DNS for 99entertainment.dk

DNS is on Cloudflare (nameservers already set), and **hosting stays on GitHub Pages**. Cloudflare is used as the DNS provider only.

1. **Verify the domain on GitHub first.** Go to github.com → your Settings → Pages → Verified domains → Add `99entertainment.dk`. Add the `TXT` record it shows (`_github-pages-challenge-rodriguez594`) in Cloudflare DNS. This stops anyone else from claiming the subdomains.
2. In Cloudflare DNS, delete any placeholder `A`, `AAAA` or `CNAME` records on `@`, `www`, `prod` or `admin`. Don't touch `MX`/`TXT` email records.
3. Add the records below with **Proxy status: DNS only (grey cloud)**. GitHub must see its own IPs to issue the HTTPS certificate.

- `A` records for `99entertainment.dk` → 185.199.108.153, 185.199.109.153, 185.199.110.153, 185.199.111.153
- Optional IPv6: `AAAA` records for `99entertainment.dk` → 2606:50c0:8000::153, 2606:50c0:8001::153, 2606:50c0:8002::153, 2606:50c0:8003::153
- `CNAME` records for `www`, `prod` and `admin` → `rodriguez594.github.io`

Then in each repo, go to Settings → Pages:
- **Source:** `main` / root for prod and admin, "GitHub Actions" for live.
- Check the custom domain.
- Tick "Enforce HTTPS" once the certificate is issued.

### If we ever move: GitHub Pages vs Cloudflare Pages (researched Oct 2026)

| | GitHub Pages | Cloudflare Pages |
|---|---|---|
| Domains | 1 custom domain per repo | Up to 100 per project. A branch can get its own domain, e.g. `prod.` → `main` |
| Server code (the push button) | None, static only | Pages Functions, 100k requests/day free |
| Bandwidth | 100 GB/month soft limit | Static requests are free and unlimited |
| Site size | 1 GB | 20,000 files, 25 MiB per file |
| Builds | 10/hour soft limit | 500/month on the free plan |
| Headers / redirects | Can't set them. Cache is fixed at `max-age=600` | `_headers` and `_redirects` files, e.g. `noindex` on prod and long caching |
| Business use | Docs say it isn't for running an online business, e-commerce or SaaS | Allowed |
| Catch | Nothing new to learn | The domain's nameservers must be on Cloudflare (needed for the root domain) |

Notes:
- Cloudflare now points *new full-stack* projects to **Workers** (with static assets) rather than Pages. Pages is not deprecated and is fully supported.
- If we move, use Pages, because "domain per branch" (prod/live) is built in there. Moving a plain static site from Pages to Workers later is simple if it's ever needed.
- The branch-domain trick only works with a proxied Cloudflare DNS record. With external DNS, `prod.` silently shows the live branch. That's another reason the nameservers must be on Cloudflare.

## 3. Backend: Supabase + video hosting

- **Supabase** (the same account as 99MS/99RIM is fine, as a new project or a new schema).
  - **Auth:** email-link login, the same as 99MS. An `admins` table holds the allowed emails.
  - **Database:** the `videos` table and the `pushes` log.
  - **Row-level security:** anyone can read `status = 'live'`, and only admins can read or write anything else.
- **Video hosting: Bunny Stream** (EU-based, very cheap, built for this).
  - Upload straight from the admin page with resumable uploads, so big files survive a bad connection.
  - It automatically creates adaptive streaming (HLS), a poster frame, and an animated preview for hover thumbnails.
  - Drafts use token-signed URLs.
  - **Alternative:** Cloudflare Stream (also fine). We don't use YouTube/Vimeo embeds, because their players and branding break the custom player design.

**Video record.** Each video stores:

| Field | Notes |
|---|---|
| `title` | |
| `artist` | |
| `category` | music video / promo / commercial / showreel |
| `year` | |
| `release_at` | Optional. A video can go live automatically at release time |
| `duration` | |
| `bunny_video_id` | |
| `poster` | |
| `preview` | |
| `credits` | List of role + name: Director, DOP, Edit, Colour… |
| `featured` | Shown on the Home reel |
| `order` | |
| `status` | `draft` → `ready` → `live`, or `hidden` |

---

## 4. Look and feel: 99 family × DON PROD structure

From the 99 repos:

- **Colours:** white on black or dark grey.

  | Token | Value | Use |
  |---|---|---|
  | `--bg` | `#000` | Video pages (black makes footage pop) |
  | `--panel` | `#1f2124` | Dark grey for contact, admin and panels (same as 99MS/99RIM) |
  | `--fg` | `#f4f5f6` | Text |
  | `--dim` | `#8a8f96` | Secondary text |
  | `--faint` | `#3a3d42` | Hairlines |

  The yellow `#e8c36a` is used only for the PROD badge, as on 99RIM. There's no other accent: the footage brings the colour.
- **Type:**
  - HUD and UI: **Share Tech Mono**, the same mono as 99MS/99RIM. It fits DON PROD's monospace HUD perfectly.
  - Labels: IBM Plex Sans.
  - Big titles: an ultra-condensed display face such as Anton, picked to sit next to the logo.
- **Logo:** the "99" numerals, saved as `assets/logo-white.svg` (main) and `assets/logo-black.svg`.
- **Name in the HUD:** `99ENT` (as on 99MGMT and @the99ent). It sits top-left where DON PROD has `DN-PRD`.
- **Language:** EN/DA switch with a flag, like 99MGMT.
- **Family link:** small footer links to 99MGMT, 99MS, 99RIM and 99WRLDWIDE.

The DON PROD structure we adopt:

- **Viewfinder corner brackets** around the focused video
- **Tick-mark gauges and scrubber**
- `[01]` indices and timecode
- **Ultra-condensed titles** with the artist name opposite
- A tracklist-style **archive** with a distorted hover preview

---

## 5. Pages

```
99entertainment.dk/
  /                Home: 99 logo, then a vertical video reel
  /work/<slug>     Video player page
  /archive         Every video as a list, with filters
  /contact         Contact, showreel, BTS photos, socials

admin.99entertainment.dk/
  /                List of videos (drag to reorder, status chips, Hide)
  /new, /<id>      Upload and edit form with live preview of the reel card
  /pushes          Push history and Undo
```

### Home
- 99 logo large, then a vertical reel. The focused video plays muted inside the brackets, and the ones around it shrink and fade.
- Left edge: a tick gauge showing where you are in the reel.
- Bottom HUD: `CATEGORY · TITLE · ARTIST · 01`. The text "scrambles" when the focus changes.

### Player
- Full-bleed video in brackets.
- Side controls: `FULLSCREEN`, `SOUND`, `PLAY`.
- A tick-mark scrubber, a huge condensed title, and the artist on the right.
- Credits list, then next/prev.
- Keyboard: space, ←/→, M, F.

### Archive
- Rows as `[01] TITLE · ARTIST · CATEGORY · 03:06`.
- The hovered row's index and duration invert.
- A floating preview has a CRT-style warp (WebGL). It falls back to a plain preview on phones and with reduced motion.
- Filters: `All [n] · Music video [n] · Promo [n] · Commercial [n]`.

### Contact
- Panel grey instead of black.
- Showreel tile, a big line (e.g. **"PRESS RECORD."**, to be decided), and Copenhagen/your city with coordinates and local time.
- BTS carousel `001 / 0NN`, a rotated scrolling "99 ENTERTAINMENT" wordmark wall, and links: Instagram @the99ent, TikTok, YouTube, email.

### Global
- Custom crosshair cursor, a loader that counts `00 → 99`, and smooth page transitions (the reel card expands into the player).
- Colour/mono toggle.
- Respects reduced motion. Mobile gets its own layouts, not a squeezed desktop.

---

## 6. Tech (no build step, like the family)

- **Code:** plain HTML, CSS and JS, with libraries from a CDN:
  - GSAP for motion
  - Lenis for smooth scroll
  - hls.js for streaming
  - OGL for the CRT shader
  - supabase-js
- **Repo layout:**

  ```
  99entertainment-prod / 99entertainment:
    index.html  work.html  archive.html  contact.html
    styles.css  js/  assets/
    supabase/functions/  (push-live, sign-upload: server side, like 99RIM's rim-ai)
  99entertainment-admin:
    index.html  admin.css  admin.js
  ```

- **Cache-busting:** `?v=<commit>` on deploy, like 99MGMT.

---

## 7. Build order

| # | Step | You can see |
|---|---|---|
| 1 | Create the prod and admin repos + DNS + Pages (§2): the three subdomains serving a placeholder, the PROD badge, `noindex` on prod | All three URLs work |
| 2 | Public site layout with dummy videos: Home reel, Player, Archive, Contact | Click through on prod |
| 3 | Supabase + admin login + video table; site reads from the database | Post a test entry in admin and see it as DRAFT on prod |
| 4 | Bunny Stream upload in admin, plus HLS playback, posters and previews | Upload a real video |
| 5 | **Push live** button, push log, Undo, instant Hide | Push and see it on 99entertainment.dk |
| 6 | Motion: loader, scramble HUD, transitions, cursor, CRT preview | The "DON PROD feel" |
| 7 | Polish: mobile, EN/DA, accessibility, speed, social preview images | Launch |

---

## 8. Needed from you

1. **DNS:** the records in §2. Also create the empty repos `99entertainment-prod` and `99entertainment-admin`, or let me know if I should.
2. **Admin emails** that may log in and push.
3. **Videos to start with:** the files (or links), plus title, artist, category, year and credits.
4. **Contact details:** email, socials, city.
5. **Supabase:** reuse the 99MS project, or a new one?
6. **Bunny Stream account** (I'll write the setup steps). Cloudflare Stream is the alternative.

---

## 9. Later

- Music production work: audio player with a waveform, and an "Audio" filter in the archive.
- Booking form, and service pages for the four 99ENT services.
- View counts per video in admin.
