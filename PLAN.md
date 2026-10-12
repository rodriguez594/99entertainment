# 99ENTERTAINMENT — Website Plan

The production arm of 99: music production, video editing, direction, post.
Reference: [donprod.uk](https://donprod.uk/) (DON PROD / "Double or Nothing Productions", built by @danny.lines).

> **Inspired by, not copied.** We take the *system* (a dark, cinematic, monospace HUD around
> full-bleed work) and build our own identity on top of it: our own logo, type pairing,
> wordmark pattern, copy and motion details. No DON PROD assets, copy or code.

---

## 1. What makes the reference work (teardown from screenshots)

| View | What it does | Steal the idea | Make it ours |
|---|---|---|---|
| **Home** (shot 1) | Black page, a large logo, then a **vertical reel of video thumbnails**. The focused clip sits in **corner-bracket "viewfinder" frame marks**; clips above/below get **smaller and dimmer**. The cursor is a crosshair. A **tick-mark scroll gauge** sits on the left edge. The bottom bar shows `CATEGORY · TITLE · ARTIST · 01`. | Scroll-snapped reel with a focus state, viewfinder brackets, HUD footer that updates on focus | "99" counter motif: the index is shown as `01 / 99`-style frame numbers, and the HUD reads like a camera/DAW readout |
| **Project player** (shot 2) | A full-bleed video inside the same brackets. Rotated side labels read `FULLSCREEN` / `SOUND` / `PLAY`. Below it is a **tick-mark scrubber** and a **huge condensed title** with the artist on the right. | Tick scrubber, vertical side controls, giant condensed type | For music projects, the scrubber becomes a **real audio waveform**, and the player handles audio-only work (beats, mixes) as well as video |
| **Contact** (shot 3) | Flips to **light mode**. It's a **hairline grid** with a showreel thumbnail, location and coordinates, a BTS photo carousel (`002 / 013`), the giant line `READY WHEN YOU ARE.`, social links, a **rotated repeating wordmark pattern**, `MODE: COLOUR`, a spaced-out wordmark, and emoji reactions (🔥 👎) with ©. | Light/dark inversion on contact, hairline grid, wordmark wallpaper | Our own line (e.g. `NINETY-NINE TIMES OUT OF A HUNDRED.` / `PRESS RECORD.`) and a "99 ENTERTAINMENT" wordmark wallpaper in our display face |
| **Archive** (shot 4) | A full **tracklist table**: `[01] TITLE · ARTIST · 03:06`. The hovered row gets inverted boxes on the index and the duration. A **hover preview video with a CRT/warp distortion** floats centre-screen in brackets. Filters at the bottom: `All [29] Music [27] Commercial [2]`. There's an RGB "colour mode" dot. | Tracklist archive, WebGL hover preview, filter counts | Filters match our services: `All · Music Video · Production · Edit · Commercial`. Rows can be **audio** (beat/mix) with a waveform preview instead of video |

**Global system:** a `DN-PRD`-style code mark top-left, `CONTACT` / `ARCHIVE` top-right, monospace caps everywhere, near-black background, 1px hairlines, a custom cursor, and a colour/mono mode toggle (videos go greyscale in mono).

---

## 2. Sitemap

```
/                 Home: vertical reel (featured work)
/work/[slug]      Project player (video or audio)
/archive          Full tracklist with filters + hover preview
/contact          Light-mode contact page, showreel, BTS carousel
/services (opt.)  Music production · Video editing · Direction · Mix/Master
```

Navigation is persistent. Page changes use **transitions** (bracket frame expands from the reel item into the player), not hard reloads.

---

## 3. Brand system for 99ENTERTAINMENT

- **Code mark (top-left):** `99-ENT` (our equivalent of `DN-PRD`).
- **Logo:** a custom "99" mark in a heavy, slanted, rounded display style, used big on Home and small in the loader. *Needs the client's logo or a commission.*
- **Type**
  - UI / HUD: monospace, caps, ~12–13px, letter-spaced. Candidates: *JetBrains Mono*, *IBM Plex Mono*, *Geist Mono* (all free).
  - Display: ultra-condensed grotesk for titles. Candidates: *Anton* or *Bebas Neue* (free), *Druk Condensed* or *Monument Grotesk Compressed* (paid).
  - Wallpaper pattern: the logo face, or a blocky rounded display face.
- **Colour:** `#000` / `#F4F4F4` plus one **accent** for 99 (TBD, e.g. signal red or tape-orange), used only for the playhead, the active index and the REC dot.
- **Motifs:** viewfinder corner brackets, tick-mark gauges, frame counters, a REC dot, waveform lines, `[01]` bracket indices, and timecode `00:03:06:12`.

---

## 4. Key interactions (build list)

1. **Loader:** a counter from `00` to `99`, then the logo, then the reel fades in.
2. **Vertical reel (Home):** smooth scroll (Lenis) plus snap. The focused item scales to 100% with brackets, and neighbours shrink to ~85% at 30% opacity. Muted autoplay applies only to the focused clip. The HUD footer text "scrambles" to the new title/artist/index.
3. **Left tick gauge:** shows scroll progress through the reel. It's clickable and draggable.
4. **Custom cursor:** a crosshair over media, a dot elsewhere, and `PLAY` / `VIEW` labels on hover.
5. **Reel → Player transition:** a FLIP of the focused thumbnail to full-bleed, brackets animate to the viewport corners, and the title slides up in the display face.
6. **Player:** a tick or waveform scrubber, `SOUND` / `FULLSCREEN` / `PLAY` side controls, keyboard controls (space, ←/→, M, F), and next/prev project.
7. **Archive hover preview:** a WebGL plane with a CRT shader (barrel warp, scanlines, RGB split, horizontal smear on movement). It follows the hovered row. Row index and duration invert.
8. **Archive filters:** live counts, and rows animate out and in.
9. **Contact:** dark→light inversion on enter, a rotated, endlessly scrolling wordmark pattern, a BTS carousel `001 / 0NN`, local time and coordinates, and a 🔥 / 👎 reaction counter (stored server-side as a fun stat).
10. **Colour / mono toggle:** a CSS filter on all media, persisted in localStorage.
11. **Audio support (our addition):** beats and mixes play through the same player with a waveform (precomputed peaks JSON), plus a mini "now playing" HUD that persists across pages.

Accessibility: honour `prefers-reduced-motion` (no smooth scroll, shader, or scramble), add real `<button>`s and focus states, add captions where possible, keep keyboard nav through the reel, and never autoplay sound.

Mobile: a single-column reel with tap-to-play, archive rows without the hover preview (tap opens the project), and a simplified player (the Awwwards entries show DON PROD has dedicated mobile player and project layouts, so this needs real design time, not just an afterthought).

---

## 5. Tech stack (recommended)

| Concern | Choice | Why |
|---|---|---|
| Framework | **Next.js (App Router) + TypeScript** | Routing, persistent layout (player/HUD survive navigation), image/SEO, easy Vercel deploy |
| Styling | **Tailwind CSS** + CSS variables for theme/mode | Fast HUD layouts; light/dark/mono via vars |
| Motion | **GSAP** (+ Flip, SplitText/ScrambleText) + **Lenis** | Industry standard for this style of site |
| WebGL | **OGL** (lightweight) or three.js | CRT hover shader, optional reel distortion |
| Video hosting | **Mux** (or Cloudflare Stream) | HLS adaptive streaming, auto thumbnails/preview clips, analytics. Don't self-host big MP4s |
| Audio | Mux audio or R2/S3 + **wavesurfer.js** / precomputed peaks | Waveform scrubber |
| CMS | **Sanity** | Projects editable without code: title, artist, category, duration, credits, video, BTS images |
| Hosting | **Vercel** | Previews per branch |
| Forms | Resend (email) or a simple mailto at first | Contact / booking |

Content model (`project`): `title, slug, artist, category[music-video|production|edit|commercial|other], year, duration, mediaType[video|audio], muxPlaybackId | audioFile, peaks, previewClip, poster, credits[{role,name}], featured(bool), order`.

---

## 6. Phases

| # | Phase | Output |
|---|---|---|
| 0 | **Inputs** | Logo, accent colour, project list (title/artist/category/duration), video and audio files, BTS photos, socials, email, location |
| 1 | **Design** | Figma frames for Home, Player, Archive, Contact (desktop and mobile), plus motion notes |
| 2 | **Scaffold** | Next.js, Tailwind, fonts, global HUD layout, routes with placeholder data |
| 3 | **Core pages** | Static versions of the reel, player, archive and contact with real layout |
| 4 | **Motion** | Lenis, reel focus logic, cursor, scramble text, loader, page transitions |
| 5 | **Media** | Mux integration, player, waveform/audio, CRT preview shader |
| 6 | **CMS** | Sanity schemas and studio, wired into the pages |
| 7 | **Polish** | Mobile, reduced-motion, a11y, performance (lazy video, poster frames, LCP), SEO/OG images |
| 8 | **Launch** | Domain, analytics, handover doc on adding new projects |

Suggested first build milestone: **phases 2–3 with dummy data** so the layout can be seen and clicked before the media pipeline exists.

---

## 7. Open questions for 99

1. Do we have a 99 / 99ENTERTAINMENT **logo** and an **accent colour**, or do we design them?
2. How much **work** is ready: how many videos and tracks, and do we have the source files?
3. Is **audio work** (beats/mixes for artists) shown publicly, or is it video only like DON PROD?
4. Do we want a **services / booking** page, or keep it portfolio-only with contact?
5. **Domain** (e.g. 99ent.co.uk / 99entertainment.com)?
6. Are there **other 99 arms** (label, clothing, etc.) we should link to from the nav?
7. Is a CMS needed (will non-devs add projects), or is a JSON file in the repo enough for v1?
