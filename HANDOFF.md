# Handoff — Apromore × Salesforce Process Intelligence deck

Context for continuing work in a fresh session. Read this first.

## What this project is
An interactive HTML pitch deck ("Apromore × Salesforce × QBE · Process Intelligence").
Pure static HTML/CSS/JS — no build step, no frameworks. The visual quality bar is
`process-reality.html` (canvas particle animation) — match that polish.

## Key files
- **`foundation.html`** — the main multi-slide deck. Arrow keys / on-screen ◀▶ navigate
  slides. Slides are `.slide` divs; nav controller is the inline `<script>` near the
  bottom (`go()`, `advance()`, `reverse()`, message listener).
- **`process-intelligence.html`** — *where the recent work is.* Standalone "The layers of
  Process Intelligence" page, embedded as **slide 5** of the deck via
  `<iframe src="process-intelligence.html?embed=1">`. Self-contained (no libs), uses
  `tokens.css`. This is where the layer animations live.
- **`process-reality.html`** — embedded as slide 4; the visual bar / interaction-model
  reference.
- **`blackbox-preview.html`** — standalone preview of the Task Mining "black box" animation.
- **`tokens.css`** — Salesforce design tokens (colours, fonts). Always pull colours from here.
- `components.css`, `animation.css`, `animation.js`, `animation-interactions.css`,
  `feedback-widget.js` — deck-wide styles/behaviour (mostly untouched recently).
- `index.html` — redirects the Pages root to `foundation.html`. `.nojekyll` present.
- `assets/` — logos, fonts (Salesforce Sans, Avant Garde display), icons.

## Hosting & GitHub
- Repo: **`scottt1234/deck-foundation`** (public). GitHub Pages is enabled and serves from
  the **`main`** branch.
- Public URLs:
  - Deck: `https://scottt1234.github.io/deck-foundation/`  (or `/foundation.html`)
  - Layers page: `https://scottt1234.github.io/deck-foundation/process-intelligence.html`
- **The user reviews on the live URL, so changes must reach `main` to be visible.**
- **Cache:** Pages' CDN + browser cache the file. To view fresh, append a cache-buster
  (`?v=9`, bump the number) and/or hard-refresh. If the user says "it's not updating",
  it's almost always cache.

## Git workflow (what works here)
- Develop on the feature branch **`claude/ecstatic-franklin-Okkpj`** (confirm the active
  branch in your session prompt — it may differ). Commit each change as its **own commit**
  (the user values per-change reversibility via `git revert`).
- Only push to `main` when the user explicitly says "push" / "push to main".
- **The user sometimes commits directly to `main`** (e.g. deleted an old slide, edited hero
  copy). So ALWAYS sync before merging or you'll clobber their work:
  ```bash
  # on the feature branch, after committing:
  git push -u origin <feature-branch>
  git fetch origin main
  git log --oneline HEAD..origin/main      # see what they added; don't lose it
  git checkout main
  git reset --hard origin/main
  git merge <feature-branch> --no-edit
  git diff <feature-branch> HEAD --stat    # expect empty (= identical trees)
  git push origin main
  git checkout <feature-branch>
  git merge origin/main --no-edit          # resync branch so it's not stale
  ```
  Recent merges have been clean because recent edits were in different files
  (`process-intelligence.html` vs the user's `foundation.html` edits). If both sides
  changed the same file, take care: keep their hunks, add yours.
- Commits are **SSH-signed automatically** (key is configured). The stop-hook may warn that
  commits are "Unverified" — this is a **false positive in this sandbox**: the signing tool
  only *signs*, it can't *verify* locally (no allowedSignersFile), so `git verify-commit`
  reports N. The commits ARE signed and show **Verified** on GitHub. **Do NOT** force-push to
  "fix" it.
- A `git push` retry-with-backoff convention is in the session instructions; network here is
  reliable so usually one push works.
- GitHub MCP tools (`mcp__github__*`) are available (scope restricted to this repo). Don't
  open PRs unless asked.

## How to preview/verify visually (important — do this before claiming "done")
There's no browser by default. Use headless Chrome via puppeteer-core:
```bash
# one-time per container:
npx -y puppeteer@23 browsers install chrome
# -> /root/.cache/puppeteer/chrome/linux-131.0.6778.204/chrome-linux64/chrome
cd /home/user/deck-foundation && npm install puppeteer-core@23   # gitignored
python3 -m http.server 8099 &                                     # serve the repo
# run a script (keep it in /tmp; resolve modules from the repo):
NODE_PATH=/home/user/deck-foundation/node_modules node /tmp/shot.js
```
Script pattern: launch with `executablePath` above + `--no-sandbox`; `setViewport
{width:1600,height:900}`; `goto('http://localhost:8099/...')`; drive with
`page.keyboard.press('ArrowRight')`; `page.screenshot({path:'/tmp/x.png'})`. Then view the
PNG with the Read tool. You can also `page.evaluate(...)` to measure element geometry /
computed styles (used heavily to verify text fits, alignment, animation timing, etc.).
- **`node_modules/`, `package.json`, `package-lock.json` are gitignored** — never commit them.
- Always validate inline JS before committing: read each `<script>` block and `new Function(s)`.

## process-intelligence.html — architecture
- **Deck embed handshake** (so the layer count lives entirely in this file; no
  `foundation.html` change needed to add/remove layers):
  - Parent → iframe: `deck-enter {dir:'fwd'|'rev'}`, `deck-advance`, `deck-reverse`, `replay`.
  - iframe → parent: `deck-next`, `deck-prev` (when stepping past first/last step),
    `embed-ready` (on load, so parent re-sends `deck-enter`).
  - Parent code: `isLayers()`, `postIframe()`, the `advance/reverse` branches, the message
    listener, and `#replay-embed` button in `.slide-controls`.
- **Steps:** 0 = overview; 1..6 = the six layers; 7 = Conversational Interface (the pillar).
  `goTo(step)`, `render()`, `showBig(step)`. The caption (eyebrow/title/sub) **bounces**
  (`piTitleIn`) on every transition.
- **Layers data:** `LAYERS[]` + `FOUNDATION`. Each big card is a swappable **animation slot**:
  `anim:'blackbox'` → Task Mining animation; `anim:'convo'` → Conversational Interface
  animation; otherwise a clean placeholder. To build a new layer animation, give that layer
  an `anim` key, add a `buildX()` / `playX()` and wire it in `showBig`'s activate/close
  branches (mirror how `convo`/`blackbox` are handled, incl. teardown of timers).
- **Embed vs standalone:** `?embed=1` adds `body.embed`, which hides the iframe's own brand,
  bottom nav and progress (the deck supplies that chrome). Replay button: standalone = the
  iframe `#nav` ↻ button; embedded = the deck's `#replay-embed` (posts `replay`).

## Design conventions (match these)
- Background: the deck's dark gradient (copy the `body` background stack). Palette from
  `tokens.css`: `--sf-blue-l` `#00B3FF` (light-blue highlight), `--sf-blue-m` `#066AFE`,
  cloud blue `#8fb8ff`.
- **Content width must match the deck**: column `max-width:1080px`, `padding:0 48px` → 984px
  content band. Caption top padding **56px** (matches process-reality). This keeps slide
  transitions seamless.
- **Cards** match deck slides 2/3: `background:rgba(255,255,255,0.05)`, `1px` border
  `rgba(255,255,255,0.12)`, `8px` radius, **2px** top accent `rgba(143,184,255,0.6)`.
  The big animation card uses the slide-8 "clean" look: electric-blue accent
  `rgba(6,106,254,0.42)`, **no resting shadow**.
- Card titles **white**, numbers **blue** (`--sf-blue-l` @ 0.55), text **centred**, no tag
  pills. Narration text uses `--font-display`.
- **Animations:** honour `prefers-reduced-motion`. Use the **Web Animations API**, NOT SMIL
  `<animateMotion>` (SMIL renders inconsistently across browsers/OS). If a token must follow
  an SVG path, sample the same bezier for its WAAPI keyframes so they coincide.
- Play animations on card open; replay on re-entry; **tear down timers on close** (store
  timer ids on the scene element, clear them in the stop function).

## Conversational Interface animation (current, step 7) — done
Narrated sequence (timings in `playConvo`, typing speed `CPS=67`):
1. Blank stage → "It used to take an **expert** to get an answer" types in ("expert" light blue).
2. Two experts appear (cyan) with a soft **spotlight** pool.
3. Line **wipes** away (clip-path).
4. Experts slowly **shrink & fly** into an **organic seeded crowd** that slowly fades in,
   while "Now **anyone** can ask for anything." types ("anyone" light blue).
5. "In their own words." types on the right (light blue, same size/align as left narration).
6. Questions stream in (rolling feed, instant-answer ✓). On each question, a **white** flash
   on a random crowd person + a token travels along a dashed line to the bubble
   (crowd→question "pulse"). White (not blue) so it's distinct from the cyan experts.
- Crowd: per-person **depth** (size/brightness/float), slightly grey `#cdd6e1`; faint
  **constellation** links; soft glow; softened gradient divider. Narration font
  `clamp(16px,2.3vw,21px)` (sized so the longest line fits one line).
- **Not yet done (optional):** #5 small asker-avatar dots on bubbles + bubble width
  variation; #6 composition tightening (kill dead space). Reversible if tried.

## Animation slot status (per layer)
State only — this doc is agnostic about which slot is worked on next; any can be built with
the swappable-slot pattern described under "architecture".
- **Conversational Interface (step 7, `anim:'convo'`)** — built & polished (detailed above).
- **Task Mining (step 2, `anim:'blackbox'`)** — present but illustrative / placeholder-quality
  (a sealed "black box" cover dissolves to reveal siloed system windows streaming into a
  unified hub).
- **Process Mining, Analysis, Recommendation, Simulation, Monitor** — clean placeholders, no
  animation yet.

## Working style the user likes
- High visual quality (process-reality bar). Verify with screenshots before saying "done".
- Small, iterative tweaks; each its own commit; keep things reversible.
- Recommend / ask rather than over-build. They'll say when to push to `main`.
- Confirm changes are reversible when trying experimental ideas.

## Recently completed (high level)
Built `process-intelligence.html` (6 slim layer cards + Conversational Interface pillar,
arrow-key walkthrough, deck embed); embedded it as deck slide 5; matched deck typography/
width/cards; added per-card animation slots + replay button; built & heavily polished the
Conversational Interface animation; updated `foundation.html` slides 1–3 copy (hero,
Transformation card, taglines) and slide-1 lead sizing. All live on `main`.
