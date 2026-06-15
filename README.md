# CAVIGLIA BBS

A retro, BBS-style interactive terminal that doubles as a personal landing page.
Single self-contained `index.html` — no build step, no dependencies (one Google
Font over CDN). Boots like a dial-up bulletin board, navigable by hotkeys or tap.

**Live:** https://pablocaviglia-uy.github.io/caviglia-bbs/

## What it is

A one-page "presentation layer" for Pablo Caviglia, designed to work for two very
different visitors at once:

- **Recruiters / non-technical visitors** get an immediately cool thing they can
  use with zero friction — a single-key (or tappable) menu that surfaces the
  bio, work focus, and LinkedIn. No puzzle to solve, no dead ends.
- **Engineers** find an optional **Door** (a real puzzle) that rewards curiosity.

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
