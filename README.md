# CAVIGLIA BBS

Pablo Caviglia's CV as a retro, BBS-style interactive terminal — a career timeline
you scan, expand, and (if you like) reach out and touch with your hands. Single
self-contained `index.html` — no build step. The base page has no dependencies
beyond one Google Font; only the opt-in **gesture mode** lazy-loads a hand-tracking
runtime from a CDN, and only if you press `H`. Boots like a dial-up bulletin board
and settles into a 2025 "liquid chrome" skin; navigable by hotkeys or tap.

**Live:** https://pablocaviglia-uy.github.io/caviglia-bbs/

## What it is

A one-page CV / career showcase for Pablo Caviglia, designed to work for two very
different visitors at once:

- **Recruiters / non-technical visitors** land on a scannable **career timeline** —
  20+ years of roles, most-recent-first, each expandable for the detail and the
  stack — plus selected projects, contact, and LinkedIn. No puzzle, no dead ends.
- **Engineers** find an optional **Door** (a real puzzle) that rewards curiosity.

Press **`T`** to flip between the 1989 BBS and a 2025 "liquid chrome" WebGL skin —
the *same* career, rendered as a bulletin-board file listing or a sleek timeline (the
1989↔2025 toggle mirrors the actual arc), and **`V`** for an opt-in webcam effect —
ASCII in retro, a refracted "liquid mirror" in modern. Both are raw inline WebGL.

Press **`H`** for an opt-in **gesture mode**: it lazy-loads webcam hand tracking
(MediaPipe, from a CDN) so you can drive the page from across the room — your
fingertip moves a cursor, **holding** over a control clicks it (dwell — a ring fills,
then it fires; no fiddly pinch), and in the modern era your hands push and ripple the
liquid chrome in real time. Strictly opt-in, torn down when off, all inference local —
nothing leaves the browser.

See `docs/PROJECT.md` for the full design rationale, the puzzle mechanics, and the
roadmap — that doc is written to let a fresh Claude Code session pick up where
this left off.

## Run locally

Just open the file:

```bash
open index.html
```

(Or serve it: `python3 -m http.server` then visit http://localhost:8000.)

## Deploy (GitHub Pages)

Already live, served from the **public** project repo
[`pablocaviglia-uy/caviglia-bbs`](https://github.com/pablocaviglia-uy/caviglia-bbs)
via Pages (Deploy from branch → `main` → `/root`) at
https://pablocaviglia-uy.github.io/caviglia-bbs/.

Because it's a *project* repo (not a `username.github.io` user-site), Pages serves
it under the `/caviglia-bbs/` path — so `og:url` and `og:image` in `index.html`
carry that subpath. To redeploy, just push to `main`; Pages rebuilds on its own.

## Structure

```
caviglia-bbs/
├── index.html        the whole app (HTML + CSS + JS, self-contained)
├── README.md         this file
├── docs/
│   └── PROJECT.md     design rationale, puzzle internals, roadmap (read this next)
└── .gitignore
```

## Notes

- The Door's access code is intentionally discoverable in the page source
  (base64 in an HTML comment) — that's the puzzle, not a leak.
- Respects `prefers-reduced-motion`. Keyboard-navigable. Works on mobile (tap).
- The camera (`V`) and gesture mode (`H`) are **strictly opt-in** and never
  auto-prompted; tracks stop and the model is torn down when you turn them off.
  All webcam processing is local — no video or landmark data leaves the browser.
