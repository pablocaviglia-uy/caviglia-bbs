# CAVIGLIA BBS

A retro, BBS-style interactive terminal that doubles as a personal landing page.
Single self-contained `index.html` — no build step, no dependencies (one Google
Font over CDN). Boots like a dial-up bulletin board, navigable by hotkeys or tap.

**Live (once deployed):** https://YOUR-USERNAME.github.io/

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

For a user site served at the root of your username:

1. Create a **public** repo named exactly `your-username.github.io`.
2. Push this folder to it (`index.html` at the root).
3. Pages auto-enables for user-site repos. (For any other repo name:
   Settings → Pages → Source: Deploy from branch → `main` → `/root`.)
4. Visit `https://your-username.github.io/`.
5. Update the `og:url` meta tag in `index.html` to that final URL.

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
