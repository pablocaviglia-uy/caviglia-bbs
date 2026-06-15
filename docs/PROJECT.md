# PROJECT — CAVIGLIA BBS

Working notes and design rationale, written so a future session (Claude Code or
otherwise) can continue without re-deriving the thinking. Last updated: 2026-06-15
— the **CV redesign** (§4d): the career timeline is now the spine, one DOM rendered
in both eras, content lives in a `CV` data object. Same day: the opt-in **GESTURE
layer** (§4c). Earlier: the Door's layer 3, the Pages deploy, the OG card, the
retro/modern FX layer (§4b).

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
  `boot | home | projects | contact | door | denied | granted | bye`.
  (Was `menu | pager` before the 2026-06-15 CV redesign — see §4d.)
- Screens are functions that `clear()` then `add()`/append lines (BBS-style full redraws).
- `boot()` types the dial-up connect sequence, draws the logo, then `goHome(true)`,
  which renders the career timeline and **settles into the modern skin** (a one-time
  `FX.switchEra`). The `ATDT` line dials Pablo's **real** mobile (a `tel:` link,
  tappable to call) — intentional, not a placeholder. The number is assembled from
  `PHONE_PARTS` at runtime (mild anti-scrape); Pablo accepted the public-exposure
  tradeoff. Don't revert it to a fake 555 number.
- **Navigation** is a persistent nav strip (career / projects / contact / the door /
  log off) — single-key hotkeys **P/C/G/Q** (+ `Esc` → home) **and** clickable/tappable.
  Career (the timeline) is home. This dual input is important — don't break it.
- **Door** uses a transparent `<input id="capture">` overlaid under a visible
  `typed` span + block cursor. The input is what raises the mobile keyboard and
  what fixes the "cursor stuck on the right" bug from an earlier version. Keep this
  pattern if you touch the input.
- Content lives in a single **`CV` data object** near the top of the script
  (tagline/about/arc/ethos + the timeline + projects). LinkedIn/Bitbucket URLs are
  constants. Edit `CV` to change what the page says — the render functions are generic.
- **One interaction contract — `act(node, fn)`.** Anything actionable (nav items,
  timeline entries, project cards, the door's back line, the redial) gets class
  `.act`, `tabindex`, click + Enter/Space, and `e.stopPropagation()`. The gesture
  layer (§4c) hit-tests `.act`, so new interactive elements become keyboard- **and**
  gesture-operable for free. Use `act()` for anything clickable; don't hand-roll.
- **Event invariant (important — was a real bug).** Two *global* listeners drive
  "press a key / tap to advance": `document` keydown and `screenEl` click. They read
  the *current* `mode`. So any inner handler that **changes the mode** in response to
  a key/click (the door's Enter→granted/denied, the door's `q`/`Escape`→home, any
  `act()` activation) **must `e.stopPropagation()`** — otherwise the event keeps
  bubbling, the global handler sees the *new* mode, and advances again (e.g. ACCESS
  GRANTED rendered then instantly bounced away). `act()` stops propagation for you;
  the `screenEl` click handler also early-returns on `.act`/`a`. The global keydown
  early-returns on `#capture`, Tab, and Cmd/Ctrl/Alt chords (so Tab reaches the
  granted-screen link and Cmd+C copies). Keep all of this if you add screens.

## 4b. The FX layer — era skin + camera (the `FX` module)

Same DOM, two skins, plus an optional webcam. All in raw inline **WebGL (no libs)**.

- **Two eras**, toggled with **`T`** (or tapping the `1989/2025` pill): `retro` (the
  BBS) and `modern` (a live "liquid chrome" fragment shader behind a glass-skinned
  panel). The skin is pure CSS driven by `body.retro`/`body.modern`; the state
  machine and every screen are unchanged — modern just hides the box-drawing
  (`.deco`/`.sepln`), swaps the ASCII logo for a `.brandModern` wordmark, and
  restyles `.pick` rows. Switching runs a **circular-reveal transition** in the
  shader (a GPU rim sweeps from the toggle) synced with the CSS crossfade.
- **Camera (opt-in, never auto-prompted)**, toggled with **`V`** / the `cam`
  button. `getUserMedia` is called **only** from that explicit action; tracks are
  `.stop()`ed on toggle-off; denial/no-camera/insecure-context flash a friendly
  message and never get stuck. Two FX: **retro = ASCII webcam** (luminance→`RAMP`,
  rendered into the fixed `#ascii` <pre> background) and **modern = "liquid mirror"**
  (the webcam fed as a WebGL texture, refracted by the flow — camera is the base,
  chrome rides on top, ~92% camera so it's clearly *you*).
- **Perf (hardened 2026-06-15, after an adversarial review):** the render loop
  **skips the fragment-shader draw entirely in retro idle** (only modern/transition/
  camera draw) — the big battery win since retro is the default; `modernScene` is
  guarded by a uniform branch so it isn't computed in retro; the ASCII readback is
  throttled to ~18fps and built via array-join; the camera texture upload is
  throttled to ~30fps; DPR is capped (1.2 mobile / 1.6 desktop); the loop pauses
  when the tab is hidden. No-WebGL devices get the CSS skin only, and turning the
  camera on there auto-falls-back to retro so it never runs blind. A shader-link
  failure sets `ok=false` and hides the canvas (CSS fallback).
- **New global keys:** `T` (era), `V` (camera) and `H` (gesture/hands) are handled
  in the global keydown, but it early-returns when the target is `#capture`, so
  typing the Door answer (which contains `t` and `h`) never toggles them. Keep that
  guard.

## 4c. The GESTURE layer — control the page with your hands (opt-in)

Reach out and touch the board from across the room. A third HUD pill — **hands**,
hotkey **`H`** — turns on webcam **hand tracking**. Same opt-in contract as the
camera (§8): never auto-prompted, torn down on toggle-off, everything stays local.

- **What it does.** Your index fingertip drives an on-screen **cursor**; a **pinch**
  (thumb + index) "clicks" whatever it's over — menu rows, the era/cam/hands pills,
  the Door's back line, or any "press any key to advance" screen. So the whole
  recruiter path (About / Work / Contact / LinkedIn) becomes usable hands-free,
  across the room, with no keyboard or touch. In the **modern** era your hand(s)
  also **push and ripple the liquid-chrome shader** in real time — landmarks feed
  `uHand[2]` / `uHandOn[2]` / `uPinch[2]`; the chrome parts around your palm and a
  pinch fires a bright radial burst. Two hands on desktop, one on mobile.

- **Still self-contained.** The base page ships zero tracking code. On opt-in it
  lazy-loads **MediaPipe Tasks Vision (HandLandmarker)** via dynamic `import()` from
  jsDelivr (pinned `@0.10.35`) plus the `hand_landmarker.task` model (~7.5 MB) from
  Google's model CDN. A recruiter who never presses `H` downloads none of it. (One
  HTML file + a dynamic CDN import is still "no build step.") MediaPipe runs
  single-threaded WASM + a WebGL2 GPU delegate (with a **mandatory CPU fallback** —
  the FX canvas already holds a WebGL context, so the delegate's second context can
  fail). It needs **no COOP/COEP / cross-origin-isolation** headers, so it works on
  plain GitHub Pages.

- **Reuses the one camera.** GESTURE never opens a second stream — it borrows the
  FX module's `getVideo()`, starts the cam only if it wasn't already on, and stops
  it on exit only if *it* started it (`ownsCam`). Turning the cam off via the `cam`
  pill drops gesture mode too (it can't track without the feed). Enabling gesture
  jumps to the modern era *after* the model loads, so a failed load leaves you
  exactly where you were.

- **Perf / lifecycle (same discipline as §4b).** A second rAF loop runs
  `detectForVideo` (synchronous) only while active, **double-gated** — capped to
  ~30fps desktop / ~18fps mobile *and* skipped unless the video frame actually
  changed — and paused when the tab is hidden. The model is created **once** and
  `close()`d on toggle-off (frees the WASM heap + GPU context). `numHands` is 1 on
  mobile. With no WebGL, gesture still works as a cursor over the retro ASCII cam
  (no liquid push). Insecure context / camera denied / model-load failure each
  flash a friendly line and fall back cleanly — the keyboard + tap paths are never
  touched. `toScreenN()` inverts the shader's cover-fit + selfie-mirror so the
  cursor and the push land where your hand actually is.

- **Privacy.** Camera frames and landmarks **never leave the browser** — inference
  is 100% in-page (WASM/WebGL). The only network traffic is the two static CDN
  downloads (runtime + model), and only after you opt in.

## 4d. The career layer — the CV is the spine (2026-06-15 redesign)

The page had drifted into a Frankenstein: the *tech* (era skin, webcam, gestures, the
Door) was the most developed part and the *career* was three thin screens — backwards
for what this site is, which is **Pablo's CV**. This pass made the career the spine and
turned the cool tech into ways to *experience* it.

- **Home is the career.** Boot (retro dial-up) → `goHome(true)` → settles into the 2025
  skin and lands on a scannable **career timeline**. `T` still flips to pure 1989 BBS;
  the 1989↔2025 toggle now frames the actual arc (BBS-era Uruguay → 20 yrs J2EE →
  Android/KMP → embedded + autonomous vehicles).
- **One timeline, two skins.** `CV.timeline` renders to identical `.tlentry` DOM
  (period · company · title · headline, expand for detail + tech chips). CSS does the
  rest: **modern** = cards on a node rail (featured roles get an accent node); **retro**
  = a BBS file-listing (cyan period, `[+]/[-]`, box rail). Same for `CV.projects`
  (`scrProjects`) and contact (`scrContact`). This *is* the "one design system, two
  expressions" fix for the disconnected feel.
- **Content is data, not markup.** Everything the page says lives in the `CV` object
  (`tagline`, `about`, `arc`, `ethos[]`, `timeline[]`, `projects[]`), curated from
  Pablo's real LinkedIn in his dry voice. To update the CV, edit `CV` — never the
  render functions. Featured vs compact roles are `tier` on each entry.
- **Gestures weave in for free** because the timeline/nav are `.act` (see §4 — `act()`):
  point + pinch expands a role or jumps a section, no gesture-specific wiring.
- The **Door** (§5), `tel:` line, and the whole opt-in tech stack are unchanged — the
  Door is still the engineer's reward, reached via `[G]`.

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

**Done:** boot sequence, ANSI logo, the **career layer** (§4d — boot→modern home,
a 12-role timeline + projects + contact rendered one-DOM/two-skins from the `CV`
object, persistent P/C/G/Q nav), the Door (3-layer puzzle + granted/denied + console
shell), logoff/redial, reduced-motion + mobile support, Open Graph tags, the
real-mobile `tel:` dial line, the **FX layer** (retro/modern era skin + opt-in
ASCII/liquid-mirror webcam — §4b, perf-hardened), and the **GESTURE layer** (opt-in
MediaPipe hand tracking: cursor + pinch-to-navigate + hands push the liquid — §4c).

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
