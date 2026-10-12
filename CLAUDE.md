# 99ENT

Website for 99ENT (99entertainment.dk), the video and music production arm of 99. See PLAN.md for the full plan.

## Hosting rule (applies to all 99 sites)

- Choose **GitHub Pages first**. That's what 99MGMT, 99MS and 99RIM use.
- Moving to **Cloudflare Pages** is always an option, but only when the user asks for it. Don't propose it as the default.
- The comparison and migration steps are in PLAN.md §2.

## Where changes go

- **Prod:** every change goes to prod first, on the `main` branch of `rodriguez594/99entertainment-prod`, served at https://prod.99entertainment.dk.
- **Live:** this repo, served at https://99entertainment.dk. It's only updated by the **Push live** button on prod, which runs `sync-from-prod.yml`, or when the user says "deploy".
- **Admin:** `rodriguez594/99entertainment-admin`, served at https://admin.99entertainment.dk.
- Each repo keeps its own `CNAME` file. **Never copy `CNAME` between repos.**

## Look

- White on black (`#000`) or dark grey (`#1f2124`), the same tokens as 99MS/99RIM.
- Fonts: Share Tech Mono for the HUD, IBM Plex Sans for labels.
- Logos: `assets/logo-white.svg` and `assets/logo-black.svg`.
- Plain HTML/CSS/JS, no build step.
