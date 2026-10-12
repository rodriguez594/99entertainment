# 99ENT

Website for 99ENT (99entertainment.dk), the video and music production arm of 99. See PLAN.md for the full plan.

## Hosting rule (all 99 sites)

- Choose **GitHub Pages first**. That's what 99MGMT, 99MS and 99RIM use.
- Moving to **Cloudflare Pages** is always an option when the user asks for it.
- **Exception: 99ENT (this repo) is on Cloudflare Pages.** The user chose this on purpose, for the three subdomains and the Push live button.
- The comparison and migration notes are in PLAN.md §2.

## Where changes go (this repo, Cloudflare Pages)

- `main` branch → https://prod.99entertainment.dk. Every change lands here first.
- `live` branch → https://99entertainment.dk. It's only updated by the **Push live** button on prod (fast-forwards `live` to `main`), or when the user says "deploy".
- `admin/` folder (Pages project `99ent-admin`, from `main`) → https://admin.99entertainment.dk.
- Never commit straight to `live`.

## Look

- White on black (`#000`) or dark grey (`#1f2124`), the same tokens as 99MS/99RIM.
- Fonts: Share Tech Mono for the HUD, IBM Plex Sans for labels.
- Logos: `assets/logo-white.svg` and `assets/logo-black.svg`.
- Plain HTML/CSS/JS, no build step.
