# PROJECT — CAVIGLIA BBS

Working notes and design rationale, written so a future session (Claude Code or
otherwise) can continue without re-deriving the thinking. Last updated: handoff
from the chat session that created the project.

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
- **Menu** rows are both keyboard (single-key hotkeys A/W/C/G/Q) **and** clickable
  (so phones work without a keyboard). This dual input is important — don't break
  it.
- **Door** uses a transparent `<input id="capture">` overlaid under a visible
  `typed` span + block cursor. The input is what raises the mobile keyboard and
  what fixes the "cursor stuck on the right" bug from an earlier version. Keep this
  pattern if you touch the input.
- Content (bio, work, links) lives inline in the screen functions. LinkedIn and
  Bitbucket URLs are constants near the top of the script.

## 5. The Door puzzle (current state)

A real, working **two-step** puzzle:

1. **Read the source.** The Door says the board "keeps its keys in the walls /
   real ones read the source."
2. **Decode the key.** Near the top of `index.html` an HTML comment contains a
   base64 string: `ZGVsZXRlY29kZQ==` → decodes to **`deletecode`** (a nod to
   Pablo's "I'd rather delete code than add it"). Entering it → **ACCESS GRANTED**,
   a personal note, and the LinkedIn link.
3. Wrong answers → a friendly **ACCESS DENIED** with a nudge (never a dead end).
4. **Deeper egg:** the granted screen points at the browser console, where a
   message admits "there is no layer 3 — yet" and invites the visitor to suggest
   one when they talk. (Intentional hook.)

The answer check is plaintext (`DOOR_KEY = "deletecode"`); the base64 is the human
puzzle, not used by the code.

## 6. Status

**Done:** boot sequence, ANSI logo, menu (hotkey + tap), about/work/contact
screens, the Door (2-step puzzle + granted/denied + console egg), logoff/redial,
reduced-motion + mobile support, Open Graph tags for the LinkedIn preview card.

**Placeholders / TODO before/after publishing:**
- Set `og:url` in `index.html` to the real Pages URL once live.
- (Optional) add a real screenshot as `preview.png` in the repo and uncomment the
  `og:image` tag so the LinkedIn card has an image.

## 7. Roadmap / ideas for next sessions

- **Build "layer 3"** of the Door — the console currently admits it doesn't exist.
  Candidate concepts discussed: a console/source/network ARG (clues across
  console logs, comments, and a faked fetch); a small reverse-engineering /
  decode chain; or a "find-and-fix the race condition" challenge (plays to the
  offline-first / concurrency depth).
- **More "rooms"** in the BBS spirit: a guestbook / message board, a "door game"
  (a tiny playable thing — Pablo has built MOT/predator-prey sims and WebXR before
  and could drop a real mini-game behind `[G]`).
- **Real preview image** for OG so the LinkedIn/Twitter card looks finished.
- Possibly grow the repo into a broader personal site / projects hub over time —
  this page is just the presentation layer "for now."

## 8. Constraints / principles to preserve

- Keep it **self-contained and dependency-free** (no build step). Easy to host,
  easy to read-the-source (which the puzzle depends on).
- Keep the **dual input** (keyboard + tap) and **no dead ends** — those protect
  the recruiter experience.
- Keep copy **specific and un-templated** — no buzzwords, no AI-generated filler.
  Pablo explicitly dislikes that register. Plain, dry, a little wry.
