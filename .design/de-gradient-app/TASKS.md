# Build Tasks: Phase 7 Review Fixes

Generated from: UX heuristics audit (2026-09-01), then design-review / web-design-guidelines
/ design-everyday-things / steve-jobs-design-review (2026-09-01, `PHASE7_REVIEW.md`).
Target: `index.html` — single file, no build step. Every task modifies it in place.

Ordered by dependency first, then severity. The status-message primitive (Foundation)
unblocks most of the Severity-3 work, so it comes first. The three other audits converged
on the same two themes (evaluation gulf → status primitive; recovery gulf → dirty/overwrite
guards) plus a batch of small accessibility and semantics fixes — see
"## Phase 7 — design-review / WIG / Norman / Jobs" below.

---

## Foundation

- [x] **Status-message primitive** — DONE 2026-09-01. `#statusLine` between `.panel__body`
  and `.export`, `role="status"` / `aria-live="polite"`. `showStatus(text, kind)` /
  `hideStatus()` / `clearErrorStatus()`; kind `info | success | error`; info/success
  auto-dismiss at 3.2s, error persists and is cleared by `clearErrorStatus()` in
  `onParamChange` / `setEffect` / `applyPreset`. New token `--color-danger #9e2b25` (the one
  non-grey chrome colour; ~7:1 on the panel). `.status` CSS with a non-colour dot signal.

- [x] **Contrast + micro-text pass** — DONE 2026-09-01. `--color-legend-muted` darkened
  `#6e6e6e` → `#666666` (from ~4.55:1 to 5.09:1 on `--color-surface`, and 4.17:1 → 4.65:1
  on `--color-surface-raised` — the second was actually below AA before). The `.select`
  chevron data-URI's hardcoded `%236e6e6e` updated to `%23666666` to track. `.switch__seg`
  idle colour switched from `--color-legend-muted` to `--color-text` (8.6:1 on raised).
  `.is-inert` no longer uses `opacity:.4` — replaced with per-element dim tokens
  (`--color-disabled` for text, `.5` opacity kept for the swatch chip since it's colour
  swatch, not text, and `--color-accent-dim` for the range track/thumb). WebGL-disabled
  `.switch__seg[aria-disabled="true"]` also uses `--color-disabled` instead of opacity.

---

## Severity 3 — major

- [x] **Still-export feedback** — DONE 2026-09-01. `exportStill()` shows "Exporting still…"
  (info), then "Saved <filename>" (success) or "Couldn't create the PNG — the frame may be
  too large for this browser." (error) on the null blob. Heavy render deferred one turn so
  the info line paints first. Upfront `eff.needsImage && !state.image` guard replaces the
  two silent inner `return`s → "Add an image before exporting." (error). _Still open:
  disabling the Still button in that state (next item)._

- [x] **Surface every silent failure** — DONE 2026-09-01. `startRecording`'s `pickMime` null,
  `captureStream` throw, and `MediaRecorder` construction throw all route to `showStatus(…,
  "error")` with actionable messages. Loop success routes to `showStatus("Saved …",
  "success")`. Zero-byte blob on `onstop` now surfaces as an error too. The `!eff.animated ||
  state.recording` early-return stays silent — it's a defensive guard the Loop button's
  disabled state already prevents from firing.

- [x] **WebGL-failure dead-end** — DONE 2026-09-01. New helper `needsGL(id)` reads from the
  `EFFECTS.source === "gradient"` data (so a future gradient-derived effect works without
  patching `buildSwitch`). `glFailed` is set before `buildSwitch` so the three WebGL segments
  never render enabled. `.switch__seg[aria-disabled="true"]` gets `opacity:.4` + `not-allowed`
  cursor; segments stay clickable so a click shows a status explaining what happened.
  Fallback effect changed from Dither (which needs an upload) to Dissolve, so the app opens
  on something already animated. Boot shows a persistent error status listing what still
  works; the `metaDims` legend keeps its short "WebGL unavailable" readout.

- [x] **Disable Still with no source** — DONE 2026-09-01. `buildExport` derives `stillReady`
  from `eff.needsImage && !state.image` and sets `still.disabled` accordingly. `exportMetaText`
  reports "Add an image to enable" for that state. `exportStill`'s upfront error message
  stays as a defensive guard (keyboard / console callers).

- [x] **Dirty-state guard on preset switch** — DONE 2026-09-01. `applyPreset` prompts with
  the exact edited-count and target-preset name when `dirtyKeys().length` and the target is
  different from the active preset. **Revert intentionally bypasses the guard** — Revert is
  the discard action, and detecting it via `name === state.activePreset[effect]` avoids
  needing a separate code path or a flag argument. **Effect switching does not need a guard**
  — the audit implied it did, but `state.params[id]` is preserved per effect, so leaving
  Gradient with edits and coming back does not lose anything. Correction noted here for
  future audits.

- [x] **User-preset management** — DONE 2026-09-01. `saveCurrentAsPreset` now: blocks the
  name if it collides with a built-in for the current effect (`showStatus(…, "error")` since
  built-ins are immutable and this is not a save the user can rescue by confirming); prompts
  to overwrite if the name collides with an existing *user* preset; and confirms the save
  with a success status. `deleteUserPreset(name)` added — user presets carry an
  `is-user` class and a trailing `×` (`.preset__delete`, a `<span>` with `stopPropagation`
  so it does not double-fire `applyPreset`; nested `<button>` is illegal so `role="button"`
  was rejected). Keyboard equivalent: Delete / Backspace on the focused user preset chip.
  If the deleted preset was active, the active preset falls back to the first built-in.
  Confirms with `window.confirm` before deleting. The `×` is danger-red on hover for the
  destructive signal.

- [x] **Loop output size** — DONE 2026-09-02. Picked a third option over the two the
  audit listed: **fixed per-ratio loop sizes at 1080p equivalents**, decoupled from the
  still delivery size. Added `lw`/`lh` to every `RATIOS` entry (1080×1080 · 1920×1080 ·
  1080×1350 · 1080×1920) and `loopDims()` reads them. `startRecording` now swaps the
  preview buffer (`previewW`/`previewH`) to loop dims *before* `captureStream` so the
  video track locks to the loop size; `finishRecording` restores. `onResize` short-
  circuits during recording so a window resize can't clobber the buffer mid-capture.
  `exportMetaText` prints the fixed loop dims. Verified: meta lines correct at all four
  ratios, buffer + `glview` canvas swap to loop dims on start (1920×1080 for 16:9) and
  restore afterwards. Not verified in headless: end-to-end file production, because
  bundled Chromium's SwiftShader doesn't reliably feed frames into `captureStream` — the
  zero-byte path fires "Recording produced no data" rather than silence. Still needs a
  real-Chrome run (STATUS.md unverified item, same as before).

---

## Severity 2 — minor

- [x] **Per-section hint text** — DONE 2026-09-02. `makeSection(legend, hint)` renders an
  optional `.section__hint` `<p>` under the legend. Hints added on the sections most likely
  to confuse a non-coder: Noise (Strength vs Density), Dissolve, Edge, Motion (dissolve),
  Tone (dither + ascii). Data-driven from `EFFECTS[…].sections[i].hint`.

- [x] **Reason on dimmed controls** — DONE 2026-09-02. `activeWhen` now takes an optional
  `reason` string ("Set Colors to 3 to use", "Set Colors to 2 per side to use"). The reason
  is written to `data-active-reason` on the control wrapper in `buildControls`, and
  `syncControlStates` copies it to `title` when the control is dim / clears it when active
  — so a greyed control's tooltip explains what to change.

- [x] **Label ratios by purpose** — DONE 2026-09-02. Each `RATIOS` entry gets a `purpose`
  string (square, video, portrait, vertical video). Applied as `title` on each ratio
  segment button — surfaces on hover / long-press without crowding the tight 4-segment
  row. `buildExport` reads `RATIOS[i].purpose`.

- [x] **Editable numeric values** — DONE 2026-09-02. `.ctrl__value` is now a real
  `<input>` styled like the old readout. Focus selects the current value; Enter or blur
  parses, clamps to `[min, max]`, snaps to `step`; Escape restores; invalid input reverts.
  Suffix (e.g. "s") is stripped before parse, then re-added. Range drag no longer
  overwrites the display during a focus (caret preserved). `tabular-nums` + fixed 56px
  width kept, so digits don't jitter.

- [x] **Persistent hex-field affordance** — DONE 2026-09-02. `.swatch__hex` gets a
  baseline `--color-seam` underline at rest. Hover darkens the line to
  `--color-legend-muted`, focus turns it accent-ink. `.ctrl__value` shares the same
  pattern.

- [ ] **Touch-target pass** — INTENTIONALLY DEFERRED. The brief says "desktop-first — this
  is a tool used on a real screen, and shader tuning on a phone isn't a use case worth
  designing for." Reopen if a mobile use case appears.

- [x] **Spacebar play/pause** — DONE 2026-09-02. `keydown` listener at document level
  routes Space to `togglePlay()`. Skipped when focus is in an `<input>`, `<textarea>`,
  `<select>`, `<button>`, or contentEditable, so it doesn't hijack the value fields, the
  effect select, or the transport button. Guarded on `!eff.animated` and `state.recording`,
  same as the click handler. `preventDefault` on Space so the page doesn't scroll.

- [x] **Large-image guard** — DONE 2026-09-02. `loadImageFile` warns via `showStatus(…,
  "info")` when the pixel count is above 12 MP ("Large image (14 megapixels) — Dither and
  ASCII exports may take a moment."). Also converted the silent-early-return on a non-image
  file into an error status.

---

## Severity 1 — cosmetic

- [x] **Naming consistency** — DONE 2026-09-02. Unified on **Cell size** for grid-of-cells
  effects (Dots, Chars, ASCII) and **Pixel size** for pixel-block effects (Dither,
  Dissolve). Two distinct concepts named consistently, per-effect meaning preserved.

- [x] **Resolve "Gradient colors" collision** — DONE 2026-09-02. `colorMode` option label
  `Gradient colors` → `From gradient` (paired with `Ink & paper` as the two colour
  sources). Select label `Gradient colors` → `Colors` (matches the Gradient effect's own
  label). No more duplicate strings in the same effect's panel.

- [ ] **Stabilise the primary button** — INTENTIONALLY DEFERRED. The primary style moves
  to the applicable-and-non-disabled action, which is what the button does in most tools.
  The audit noted it as cosmetic; button *positions* are already fixed (Still left, Loop
  right). Revisit only if it becomes a real problem in use.

- [x] **loopTime affordance** — DONE 2026-09-02. `.transport__time` gets `cursor:default`,
  `user-select:none`, and `font-variant-numeric:tabular-nums`. Reads as a status display,
  not a scrubber.

---

## Phase 7 — design-review / WIG / Norman / Jobs

Findings from the other four Phase-7 lenses not already covered above. Full write-up in
`PHASE7_REVIEW.md`.

### Must fix (accessibility — design-review)

- [x] **Heading hierarchy + landmarks** — DONE 2026-09-01. `<span class="wordmark">` →
  `<h1 class="wordmark">`; `.preview` `<section>` → `<main aria-label="Preview">`;
  section legends → `<h2 class="section__legend">` inside `makeSection` and both Export
  paths in `buildExport`. `.switch__glabel` was already gone with the variant B rewrite.
  Outline verified: one `<h1>` ("DE-GRADIENT"), seven `<h2>`s (Presets, Color, Motion, Noise,
  Finish, Import, Export). Visual is byte-identical because the classes carry the styling
  and the reset zeros heading margins.
- [x] **`<canvas>` accessible name** — DONE 2026-09-01. `#artboard` carries `role="img"`
  and an `aria-label` that `syncStage()` updates to the active effect on every switch
  ("Gradient — live preview") or to "Drop an image to dither" / "…ascii" while an image
  effect is empty.
- [x] **Contrast** — DONE, folded into "Contrast + micro-text pass" above. The extra
  `.switch__seg` idle-colour fix ships with it.

### Quick wins (WIG — batched 2026-09-01, all DONE)

- [x] **`color-scheme: light`** on `:root` — native select / color-input / scrollbar all
  render for a light theme regardless of OS dark mode.
- [x] **`<meta name="theme-color" content="#d9d9d9">`** in `<head>`.
- [x] **`preconnect` for `api.fontshare.com`** added.
- [x] **`touch-action: manipulation`** and `-webkit-tap-highlight-color: transparent` on
  `body`.
- [x] **Import field**: `type="url"`, `inputmode="url"`, `name`, `autocomplete="off"`,
  `spellcheck="false"`, placeholder now `"shadergradient.co URL…"`.
- [x] **`:focus-visible`** replaces `:focus` on `.swatch__hex` and `.input`.
- [x] **`overscroll-behavior: contain`** on `.panel__body`.
- [x] **Curly apostrophes** in the two import messages and the ignored-params copy
  (`"doesn’t"`, `"couldn’t"`).
- [x] **`.source__thumb`** `<img>` carries explicit `width={32} height={32}`.
- [x] **`translate="no"`** on the wordmark `<h1>` and on every `.swatch__hex` input.
- [x] Deleted dead tokens `--radius-lg` and `--space-1` (verified unused; parse still clean).

### aria-live (WIG — needs the status primitive or a small standalone pass)

- [ ] **`aria-live="polite"`** on: the import report (`index.html:2121`), the preset status
  note (`index.html:2199`), and the recording progress value (`index.html:2270`). If the
  status-message primitive lands first, route these through it (it should carry
  `role="status"` / `aria-live`).
- [ ] **WebGL-failure message** gets `role="alert"` and a full sentence (folds into the
  existing "WebGL-failure dead-end" task).

### Toggle control (WIG)

- [x] **Toggle label + hover** — DONE 2026-09-02. `<span>` → `<label for="…">` tied to the
  switch button's id. `<label for="">` only forwards clicks to real form inputs, not to
  `<button role="switch">`, so a click handler on the label routes to the same `toggle()`
  function. `.toggle__switch` gets a subtle `:hover` (`--color-seam` off, `filter:brightness`
  on) so it reads as interactive.

### Offline promise (Jobs fix #3 — promoted from `STATUS.md` "Next up #5")

- [x] **Embedded Quantico + Nippo as base64 `@font-face`** — DONE 2026-09-02. Quantico
  400 + 700 sourced from the portfolio repo's `public/fonts/`. Nippo 700 was **not** in
  the portfolio (the brief's assumption was wrong for that one); fetched from Fontshare
  (SIL OFL, embeddable) and cached to the scratchpad. Only weights the app actually uses
  ship — three `@font-face` rules total, no italics, no other weights. The five link/
  preconnect tags in `<head>` are gone. File is 158 KB, +46 KB from before; verified in
  a headless Chromium with `Network.emulateNetworkConditions({offline:true})`: all three
  fonts load, zero external requests, wordmark computes to `Nippo`, controls compute to
  `Quantico`.

### Error copy (Norman / Jobs)

- [x] **Import error messages carry a next step** — DONE 2026-09-01. "That URL couldn’t be
  read — paste a full shadergradient.co customize link." and "No colours or numbers found
  in that URL — check the link is from shadergradient.co/customize." The status-primitive
  messages (Batch 1) were already actionable when written.

### Loop output size (already listed under Severity 3 — decision needed)

- [ ] Confirmed by every lens as a real surprise (`review-ratio-169-desktop`: "MP4 912×514"
  vs still 3840×2160). Decide: render the loop to the ratio's real `ew/eh`, or state the
  actual pixels plainly. Don't leave it implicit.

## Cut list (steve-jobs-design-review — row 5: this cycle must remove something)

- [x] **Dropped the duplicate "add an image" button** — DONE 2026-09-02. `makeSource`
  returns `null` when there's no image, and `buildControls` now skips sections that end
  up with only a legend. The dropzone on the canvas is the CTA in that state; when an
  image is loaded, the panel Source row (thumb + Replace) appears, which is meaningful.
- [ ] **Consider cutting**: 5 charsets → 3 (Classic / Blocks / Binary; Minimal and Dots
  read as near-duplicates of Classic). "Wave scale" — a control a non-coder never touches;
  fold into presets. _Both need Daniel's call before cutting — judgement calls._

## Panel-nav restructure — DECIDED: variant B as drawn

Daniel picked **B** (dropdown) 2026-09-01, rejecting C's off-brand dark pills. B as drawn
in the artifact `630b10da` "De-Gradient Panel B" — **two-level nav**: a top segmented control
"Generative / From image" (mode tabs, accent-fill active — matches today's active language),
then the current mode's effects in a `<select>` inside that tab. A single grouped-select
was considered and dropped: it changed the structure Daniel chose without asking.

Consequences:

- **Cross-mode switching is two clicks** (mode tab → open dropdown → choose). Today it is
  one click. Accepted trade-off — the "direct manipulation" principle applies to *tuning*
  a look, not to changing which effect you're on.
- The **From-image tab always shows the dropzone** at the top when there's no image,
  regardless of which of Dither/ASCII is selected — the mode's shared prerequisite.
- **Effect state must survive the tab swap.** `state.params[id]` already stores per-effect
  parameters; the RAF `tick()` / `syncStage()` / `render()` need to keep reading the active
  effect from `state.effect` and treat the tab as a *view* of that state, not the source.

Build tasks:

- [x] **Replace `buildSwitch` with mode tabs + per-tab select** — DONE 2026-09-01. Two
  `<button class="switch__seg" role="tab">` for modes, then a native `<select class="select
  fx-select">` for the current mode's effects. `EFFECT_GROUPS` stays as the data source;
  helpers `modeOfEffect(id)` and `groupForMode(mode)` derive membership without patching.
  The dead `.switch__group` / `.switch__glabel` CSS was removed.
- [x] **Per-mode memory** — DONE 2026-09-01. `state.mode` and `state.lastEffectInMode` are
  independent of `state.effect`. `setEffect` updates all three; `setMode` looks up the
  mode's last-active effect and switches through. Verified: Generative→Dissolve, tab to
  From image (lands on last From-image effect), tab back (returns to Dissolve, not
  Gradient).
- [x] **Dirty-guard on the `<select>` change** — NOT NEEDED, correction. Effect switching
  is already non-destructive because `state.params[id]` is preserved per effect. The audit
  was wrong to imply the guard on effect switch closed a data-loss gap; it doesn't. Guard
  stays only on `applyPreset`.
- [ ] **Optional: EFFECTS-driven description line under the select.** Not built. Would
  subsume half of the Severity-2 "per-section hint text" task and mitigate the
  discoverability cost of hiding the other-mode effects. **Decide before Batch 8.**

### Reprioritised by the B decision

- [ ] **`color-scheme: light` + explicit `<select>` / `<option>` `background-color` + `color`**
  moves from "quick win" to **required** — B leans on native selects; Windows dark mode
  would render the effect picker dark-on-dark.
- [ ] **Naming consistency** ("Dot size" / "Pixel size" / "Cell size") and the
  **"Gradient colors" collision** move up from Severity 1 — effects are one dropdown deep
  now, so labels can't be reconciled by looking at them side by side.
- [ ] **Empty-state copy for the From-image tab**: the dropzone message should read the
  same regardless of which of Dither/ASCII is selected, or explain the difference — today
  the dropzone title switches between "dither" and "convert". Consider a shared "Add an
  image" title with the effect name below.

## Review gate — RUN 2026-09-02 (see `REVIEW_GATE.md`)

- [x] **Re-run /ux-heuristics** — **10/10.** All ten Quick Diagnostic rows pass; every
  Nielsen heuristic covered. Ratio purpose remains a hover-only affordance (title tooltip),
  documented and acceptable per the desktop-first brief.
- [x] **Re-run design-review** — every must-fix from PHASE7_REVIEW.md closed.
  Aesthetic-fidelity note: the persistent hex-field underlines may read a hair too strong
  on the swatch rows at desktop; if it grates, weaken the resting underline to
  `--color-surface-raised` and keep hover as `--color-seam`. Optional polish, not a defect.
- [x] **Re-check Norman score** — **10/10.** Both gulfs closed (evaluation via the status
  primitive; recovery via dirty-guard + preset delete + overwrite confirm). Constraint gap
  (dead Still) closed too.
- [x] **Re-check Jobs** — **INSANELY GREAT (9/10).** Six of seven rows pass. Row 7
  ("would the team sign it and use daily") still needs Daniel to actually open the build.

### Still open — needs Daniel

- [ ] **Open the current build by eye** and confirm the whole thing feels right.
- [ ] **Decide the two remaining cuts:** 5 charsets → 3? Fold Wave scale into presets?
- [ ] **Real-Chrome MP4 run** — capture a full loop, confirm 1920×1080 playback is
  seamless. Bundled Chromium can't produce one.

## Post-review-gate changes (2026-09-02, before ship)

Daniel corrected two things at the review gate:

- [x] **Inactive controls: hide, not dim** — the earlier 2026-08-01 "dim, not hide" fix
  reversed. `syncControlStates` now toggles `hidden` on the wrapper instead of `.is-inert`
  + `disabled`. `[hidden]{display:none!important}` added globally so class rules can't
  override. `.is-inert` CSS deleted, `activeWhen.reason` field + `data-active-reason`
  plumbing deleted. Reasoning trail preserved in the brief.
- [x] **Random button per effect** — new `↻ Random` chip next to `+ Save` in an action
  row at the bottom of the presets grid. `randomiseCurrent()` walks the effect's sections,
  rolls a new value for every parameter (except any gating select). Ranges snap to `step`,
  selects pick a random option, colours are HSL with clamped saturation (45–85%) and
  lightness (35–75%) — no muddy or garish output. After Random:
  `state.activePreset[effect]` is null, no preset chip is highlighted, the preset note
  reads "Random", and a status message prompts to Save.

## Simplify pass (2026-09-02)

Four cleanup agents (reuse, simplification, efficiency, altitude) run in parallel; findings
consolidated and applied. Runtime + visual verified after: variant B nav, hide-not-dim,
Random, editable values, and WebGL-off boot all preserved; zero console exceptions.

- [x] **Memoize `pickMime` → `MIME_CHOICE`** — was recomputed 3× per slider tick through
  `exportMetaText` → `recorderSupported` + `recorderLabel`. MediaRecorder support is a
  browser-session property; pick it once at boot.
- [x] **`refreshExportMeta` compare-before-write** — only `loopDuration` and ratio change
  the meta string, but every slider drag was writing it identically 60×/sec.
- [x] **Cache RAF-tick DOM nodes** — `loopTimeEl`, `recFillEl`, `recValueEl` cached at
  module scope like `statusEl`; text updates now guard against string equality so repeat
  writes of the same value don't hit the DOM.
- [x] **`usePreviewSize(w,h) → restore()` helper** — collapses the buffer save/set/render
  triplet that was inlined at four places (startRecording body + two catch blocks +
  finishRecording) into one closure. Removes `rec.prevPreviewW/H` fields entirely, and
  removes the "unreachable but cheap defense" guard the simp agent flagged.
- [x] **Drop `state.mode` field** — derivable via `groupOfEffect(state.effect)`. Removes
  one invariant to keep in sync (`setEffect` no longer has to touch it).
- [x] **Unify `modeOfEffect` + `groupForMode` → `groupOfEffect(id)`** — one lookup instead
  of two. Deleted the bootstrap loop that pre-seeded `lastEffectInMode` (redundant with
  `setMode`'s `|| group.ids[0]` fallback) and the unreachable `EFFECT_GROUPS[0]` fallbacks.
- [x] **`EFFECT_GROUPS` gains `id` field** — `{ id, label, ids }` per group. `state.lastEffectInMode`
  keyed by `id`, mode tabs identified by `id`, label is display-only. A label rename no
  longer strands state.
- [x] **`needsGL` as a declared flag** — `needsGL: true` on Gradient, Dots, Chars, replacing
  the `id === "gradient" || source === "gradient"` inference which would break as soon as
  a future non-GL effect was named "gradient" or a GL effect wasn't gradient-derived.
- [x] **`randomiseCurrent` derives gate-skip from `activeWhen` keys** — no more hard-coded
  `"colors"` / `"colorMode"` names. A new gating select automatically escapes the roll.
- [x] **Trim `syncControlStates`** — stale doc-block (longer than the 8-line function it
  fronted) removed; dead `activeWhen.value.split(",")` multi-state support dropped (no
  caller uses it).
- [x] **Move `hslToHex` / `randomHexColor` to the colour helpers block** — sit beside
  `hexToRgb` / `normalizeHex` where they belong, not inside preset actions.
- [x] **Extract `.field` CSS base class** — shared by `.ctrl__value` and `.swatch__hex`
  (transparent, borderless, right-aligned, tabular-num inputs with the same
  underline-at-rest / accent-ink-on-focus affordance). Each keeps only its own width /
  weight / colour overrides. ~15 lines of CSS to 3.

### Skipped (documented)

- **`wireEditableInput` helper for `makeRange` + `makeColor`.** Real duplication exists —
  both wire focus-select / Enter-commit / Escape-restore / blur-settle — but each captures
  distinct state and the mid-type write differs (colour skips writing `hex.value` to
  preserve caret; range writes `val.value` on drag to keep display honest). Two callers
  isn't enough to earn the callback plumbing; revisit if a third editable-input control
  arrives.
- **`makeSegmentedRow` for `buildSwitch` + `buildExport`.** Same story — the JS is close
  but the state each row tracks differs enough that extraction means passing 4+
  callbacks. The `.switch__row` / `.switch__seg` CSS is already shared; that's the
  meaningful reuse.
- **`makeColor` triplet through `commit()`.** False positive — chip + settle paths already
  route through `commit`; the mid-type path deliberately doesn't (comment in-source
  explains: writing `hex.value = v` mid-keystroke would throw the caret to the end).
- **`section.hint` as string.** Right-sized for what's there; upgrade to markup only if
  a hint actually needs it.
- **`applyPreset` `isRevert` name-collision detection.** Special case flagged at altitude
  but the caller-side alternative (pass a flag) is more code for one call site. Left as-is.
