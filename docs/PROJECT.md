# PROJECT — CAVIGLIA BBS

Working notes and design rationale, written so a future session (Claude Code or
otherwise) can continue without re-deriving the thinking. Last updated: 2026-06-14
— built the Door's layer 3, deployed to GitHub Pages, added the OG preview card.

**Live:** https://pablocaviglia-uy.github.io/caviglia-bbs/ (public project repo,
served at the `/caviglia-bbs/` subpath — not a user-site, so URLs carry the path).

---

## 1. Purpose

A personal landing page for Pablo Caviglia, reached from his LinkedIn (Featured
section + Contact-info website link). It exists to make a memorable, distinctive
first impression and route the right people to LinkedIn — **without** reading as
"I'm job hunting." It reads as an engineer's side project, which is also the
honest truth.

## 2. The core design problem (and the solution)

The page has to serve two audiences who want opposite things:

- A **recruiter / non-technical visitor** wants something cool and effortless.
  A hard puzzle would lock them out and make them feel stupid → they bounce.
- An **engineer / technical hiring manager** wants depth and a real challenge.
  A purely "pretty" page tells them nothing.

**Solution: layer it, with the easy win up front and the hard win optional.**
Never gate the payoff behind the puzzle.

- **Floor (everyone):** the page is just cool to watch and touch — it dials in
  like a BBS, draws an ANSI logo, and presents a menu. The aesthetic alone says
  "this person is skilled." The menu surfaces bio / work / LinkedIn with a single
  keypress *or a tap*. They leave impressed, having solved nothing.
- **Middle (curious):** small delights — the boot sequence, the redial/NO CARRIER
  flow.
- **Vault (engineers):** an explicitly-optional **Door** with a genuine puzzle and
  a rewarding payoff. Clearly flagged as "for engineers," so non-techies know it's
  fine to skip it. Always an escape back to the menu — no dead ends.

## 3. Aesthetic

BBS / ANSI dial-up era (Pablo's actual nostalgia — he was a BBS guy). Chosen over
a generic amber CRT terminal because it's more personal *and* more distinctive,
and because a hotkey/tappable menu is a lower barrier for recruiters than a unix
command line.

- CRT chrome: scanlines, vignette, subtle flicker, glow (all respect
  `prefers-reduced-motion`).
- ANSI 16-color palette used with restraint — cyan/white for structure, yellow
  for hotkeys, magenta for the Door, green for success, red for denial.
- `░▒▓█` gradient blocks + box-drawing chars for the BBS texture.
- Font: VT323 (Google Fonts).

## 4. How it works (implementation)

Everything is in `index.html`: HTML shell + `<style>` + one `<script>`.

- A tiny **state machine** drives it: `mode` ∈
  `boot | menu | pager | door | denied | granted | bye`.
- Screens are functions that `clear()` then `add()` lines (BBS-style full redraws).
- `boot()` types the dial-up connect sequence, draws the logo, then `drawMenu()`.
  The `ATDT` line dials Pablo's **real** mobile (a `tel:` link, tappable to call) —
  intentional, not a placeholder. The number is assembled from `PHONE_PARTS` at
  runtime (mild anti-scrape); Pablo accepted the public-exposure tradeoff. Don't
  revert it to a fake 555 number.
- **Menu** rows are both keyboard (single-key hotkeys A/W/C/G/Q) **and** clickable
  (so phones work without a keyboard). This dual input is important — don't break
  it.
- **Door** uses a transparent `<input id="capture">` overlaid under a visible
  `typed` span + block cursor. The input is what raises the mobile keyboard and
  what fixes the "cursor stuck on the right" bug from an earlier version. Keep this
  pattern if you touch the input.
- Content (bio, work, links) lives inline in the screen functions. LinkedIn and
  Bitbucket URLs are constants near the top of the script.
- **Event invariant (important — was a real bug).** Two *global* listeners drive
  "press any key / tap anywhere to advance": `document` keydown and `screenEl`
  click. They read the *current* `mode`. So any inner handler that **changes the
  mode** in response to a key/click (the door's Enter→granted/denied, the door's
  `q`/`Escape`→menu, a menu row's Enter/Space/click) **must call
  `e.stopPropagation()`** — otherwise the same event keeps bubbling, the global
  handler sees the *new* mode, and immediately advances again (e.g. ACCESS GRANTED
  rendered then instantly bounced back to the menu). Every such transition source
  now stops propagation; keep that if you add screens or controls. The global
  keydown handler also early-returns on Tab and Cmd/Ctrl/Alt chords so Tab can
  reach the LinkedIn link on the granted screen and Cmd+C can copy it.

## 5. The Door puzzle (current state)

A real, working **three-layer** puzzle. Layers 1–2 live in the page UI; layer 3
lives entirely in the browser console (so the recruiter path never sees it).

**Layer 1 — read the source.** The Door says the board "keeps its keys in the
walls / real ones read the source."

**Layer 2 — decode the key.** Near the top of `index.html` an HTML comment
contains a base64 string: `ZGVsZXRlY29kZQ==` → decodes to **`deletecode`** (a nod
to Pablo's "I'd rather delete code than add it"). Entering it → **ACCESS
GRANTED**, a personal note, the LinkedIn link, and a pointer at the console.
Wrong answers → a friendly **ACCESS DENIED** with a nudge (never a dead end).
The answer check is plaintext (`DOOR_KEY = "deletecode"`); the base64 is the
human puzzle, not used by the code.

**Layer 3 — reverse-engineer the seal (console-only).** On ACCESS GRANTED,
`wireShell()` runs once and attaches a `window.sysop` maintenance shell
(`help / whoami / ls / cat / open / hint`). Everything is scoped inside
`wireShell`'s closure — only `window.sysop` leaks. One file, `sysop.note.sealed`,
holds a sealed blob `WV5LU1lDR1pGTw==`. The passphrase is **not** in the source;
only the blob is. The verifier `sysop.open(pass)` computes
`seal(pass) === SIGIL`, where `seal` is a symmetric single-byte XOR (key `42`)
then base64. You solve it by reading the verifier (`sysop.open.toString()` or the
page source) and **inverting** the transform in the console:
`atob(blob)` → XOR each byte with 42 → `staysimple`. Then `sysop.open("staysimple")`
prints the final payoff (the `deletecode → staysimple` arc — both are literally
Pablo's stated ethos — plus a "say 'node 1' when you message me" handshake).

  - Solve paths, all self-contained, no dead ends: read the page source; or read
    `sysop.open.toString()`; or brute-force the 1-byte key (only 42 yields clean
    lowercase); or follow `sysop.hint()`, which explains the symmetry without
    handing over the key.
  - `open()` normalizes input (`toLowerCase`, strips non-alphanumerics) so
    `"Stay Simple"` works. `grantLayer3()` is idempotent (`opened` flag).
  - To change the answer: pick a new passphrase, recompute the blob with
    `btoa([...p].map(c=>String.fromCharCode(c.charCodeAt(0)^42)).join(''))`, and
    replace `SIGIL`. (Keep the answer `[a-z0-9]` so the XOR stays printable.)

## 6. Status

**Done:** boot sequence, ANSI logo, menu (hotkey + tap), about/work/contact
screens, the Door (3-layer puzzle + granted/denied + console shell), logoff/redial,
reduced-motion + mobile support, Open Graph tags for the LinkedIn preview card.

**Deployed:** public repo `pablocaviglia-uy/caviglia-bbs`, GitHub Pages from
`main` / root → https://pablocaviglia-uy.github.io/caviglia-bbs/. `og:url` and
`og:image` point at that subpath; `preview.png` (1200×630, the main menu) is
committed at the repo root and served alongside `index.html`.

**If you regenerate `preview.png`:** it's a headless-Chrome shot of the live menu.
Serve the folder (`python3 -m http.server PORT`), then:
`"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new
--disable-gpu --hide-scrollbars --window-size=1200,630 --virtual-time-budget=12000
--screenshot=preview.png http://localhost:PORT/`. The virtual-time budget fast-
forwards the boot animation so it lands on the menu.

## 7. Roadmap / ideas for next sessions

- **Layer 3 is built** (console reverse-engineering shell — see §5). If you want a
  *layer 4* someday, the cleanest hook is inside the shell: `sysop` could expose a
  "node 2" you connect to, or a faked `fetch` whose response carries the next clue.
  Keep the same rule — never gate the recruiter payoff behind any of it.
- **More "rooms"** in the BBS spirit: a guestbook / message board, a "door game"
  (a tiny playable thing — Pablo has built MOT/predator-prey sims and WebXR before
  and could drop a real mini-game behind `[G]`).
- Possibly grow the repo into a broader personal site / projects hub over time —
  this page is just the presentation layer "for now."

**Polish backlog** (found in a 2026-06-14 audit; none are dead ends, all are
nice-to-haves, listed worst-first):
- *Mobile door focus:* `#capture` is focused once via `setTimeout`. On phones a
  tap that misses the (invisible) input can drop focus with no visible "tap to
  type" affordance and no re-focus handler — the engineer-on-a-phone path can
  stall on the Door (still escapable via the `[Q]` line, so not a dead end).
  Fix: a `mode==='door'` click handler that re-`focus()`es `#capture`.
- *Focus after transitions:* nothing calls `screenEl.focus()` after a redraw, so
  a keyboard user who entered via tap relies entirely on the global listener
  (works, but Tab focus is inconsistent). Fix: `screenEl.focus()` at end of redraws.
- *`type()` interval:* the typewriter `setInterval` has no handle and isn't
  cancelled on navigation. Harmless today (only used in `boot()` before input is
  wired), but a latent zombie-interval if `type()` is ever used on a navigable
  screen. Fix: store + `clearInterval` in `clear()`, or guard on `node.isConnected`.
- *`scrBye` timing:* the 500ms NO CARRIER delay isn't gated by `reduce`, and the
  pending timeout isn't cleared if you redial fast — a quick keypress can leave a
  stray "NO CARRIER" line in the next boot. Fix: gate with `reduce`, store + clear
  the timeout id.
- *Caller number:* `1000+random*8999` tops out at 9998 and re-rolls every boot.
  Cosmetic; persist in `localStorage` for a stable number if desired.

## 8. Constraints / principles to preserve

- Keep it **self-contained and dependency-free** (no build step). Easy to host,
  easy to read-the-source (which the puzzle depends on).
- Keep the **dual input** (keyboard + tap) and **no dead ends** — those protect
  the recruiter experience.
- Keep copy **specific and un-templated** — no buzzwords, no AI-generated filler.
  Pablo explicitly dislikes that register. Plain, dry, a little wry.
