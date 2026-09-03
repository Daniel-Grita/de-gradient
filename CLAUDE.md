# De-Gradient

*(formerly Gritaflux — renamed 2026-09-02. Local folder `C:\Code\Gritaflux\` kept
for continuity; localStorage keys still `gritaflux.*` so saved presets survive.)*


A local, single-file browser app for making generative visuals — animated
gradients, dither fields, ASCII — and exporting them as PNGs or seamless video
loops, at a chosen aspect ratio.

**The load-bearing constraint: the owner is not a coder.** Every feature must be
reachable from the UI. Never add something that requires editing code to use.

## Running it

Double-click `index.html`. No build step, no server, no npm, no dependencies.
That is the whole point — do not introduce tooling.

## Where things live

| Path | What |
| --- | --- |
| `index.html` | The entire app. Authoritative. |
| `.design/de-gradient-app/DESIGN_BRIEF.md` | Why it is the way it is. Read before changing UX. |
| `.design/de-gradient-app/STATUS.md` | Build state, known risks, what is next. |

Design exploration lives in Paper, file `01KWYYT0M4R02GE4W4S2CA02VM`.

## Architecture

Read `DESIGN_BRIEF.md` first. The short version:

- **Effects are data.** `EFFECTS` in `index.html` declares each effect's params,
  control sections and presets; the panel builds itself from that declaration.
  **Adding an effect means adding an entry there, not writing UI code.** Wire it
  into `EFFECT_ORDER`, `EFFECT_GROUPS`, and the `render()` / `exportStill()`
  switches. One caveat: the panel clears a 900px screen only because its gaps
  were hand-tightened to off-scale values, and the Export footer carries a ratio
  row on top of its two buttons. Adding Dissolve as a sixth effect tested that
  and it held — **`.switch__row` is a flex row of `flex:1` segments, so a new
  effect inside an existing group makes the segments narrower, not the row
  taller.** Four labels sit at 70px each with no ellipsis. What would actually
  cost height is a *third* `EFFECT_GROUPS` entry, since each group is its own
  label + row. The section list is a separate budget: `.panel__body` scrolls, so
  a long effect only pushes its own controls down, not Export.
- **Two canvases.** `#glview` (WebGL) and `#c2dview` (Canvas 2D) are stacked and
  toggled; only one is visible. `activeCanvas()` decides which one export and
  video capture read from — keep it correct when adding effects.
- **The frame is the export.** Aspect ratio is a global setting (`RATIOS`,
  `state.ratio`), not a per-effect param and deliberately not stored in presets.
  `fitPreview()` sizes the frame to that ratio — largest that fits — and gives
  the backing store the same shape, so the preview is the file rather than a
  crop of it, and loop capture records exactly what is on screen. **Nothing may
  assume a square:** every draw function takes `w, h` separately, cell grids
  derive their row count from one shared cell size so cells stay square, and the
  gradient shader samples noise in short-edge units so a wide frame is a wider
  *view* of the field, not a stretched one. Export sizes per ratio are delivery
  sizes set in the brief, not a formula — see the comment on `RATIOS`.
- **The loop is exact.** 4D simplex noise, two dimensions tracing a circle over
  the loop duration. Do not replace this with a cross-fade; the seamlessness of
  every exported loop depends on it.
- **Generative effects** (Gradient, Dots, Chars, Dissolve) are animated and allow
  Loop export. **Image effects** (Dither, ASCII) are static and disable it.
- **Dots and Chars sample the gradient at cell resolution**, not full resolution.
  Preserve that — averaging a full-size render down per frame is far too slow.
- **Colour count is a control, not a constant.** Gradient/Dots/Chars take a
  `colors` select (3 or 2 — two skips the third mix in the shader rather than
  neutralising `uColor3`, which would still wash the field). Dissolve takes
  1 or 2 *per side*, the second colour scattered into the first on the same
  pixel grid with a `Mix` amount, so each field stays flat.
- **A control that does not apply is hidden.** Declare
  `activeWhen: { key, value }` on it and `syncControlStates()` toggles the
  wrapper's `hidden` attribute. `[hidden]{display:none!important}` is set
  globally so class-selector rules (`.ctrl{display:flex}`) can't override it.
  `value` may be comma-separated for several matching states. Only selects gate
  other controls, so their change handler is the only place that re-syncs; it
  toggles `hidden` rather than calling `buildControls()`, which would move
  focus off the select the user just used. **This is a reversal of the earlier
  "dim, not hide" fix** — see brief for the reasoning trail. Do not add
  `.is-inert` back; the class is gone.
- **`colors` is stored as a string** (`"3"`, `"2"`, `"1"`), because `makeSelect`
  writes `sel.value`. Numeric defaults would read as permanently dirty against
  it. `pattern: "8"` is the same shape.
- **`applyPreset` backfills from `defaults` before applying.** A user preset
  saved to `localStorage` before a param existed has no value for it, and
  without the backfill it would inherit whatever was set last — the same preset
  applying differently depending on what preceded it. Keep this when adding
  params.
- **`randomiseCurrent` skips `colors` and `colorMode`** — those two selects
  gate which controls are visible, and randomising them would churn the panel
  shape on every click. Every other param gets a fresh roll: ranges snap to
  their `step`, selects pick a random option, colours are sampled in HSL with
  clamped saturation (45–85%) and lightness (35–75%) so results stay usable.
  After Random, `state.activePreset[effect]` is set to `null` and the preset
  note reads "Random"; save if you want to keep it.
- **Dissolve is generative but not gradient-derived.** It is the one effect that
  touches neither the shader nor an upload — two flat colours and a probability
  ramp across a band. Its scatter is a deterministic per-cell hash
  (`hashCell`), never `Math.random()`: the field must be identical frame to
  frame or the whole image fizzes. Everything that moves in it — band drift,
  edge wave, cell boil — completes a whole number of cycles per loop, which is
  how it satisfies "the loop is exact" without any noise field at all.

## Design system

De-Gradient runs a **light, neutral instrument grey** theme. It descends from
Daniel's portfolio system (`github.com/Daniel-Grita/daniel-grita-portfolio`,
`src/styles/global.css`) but is no longer that system's dark palette — the
neutrals are its own and there is only one theme.

- All values live in one `:root` block. Change them there, never inline.
- Fonts: **Nippo** (wordmark) + **Quantico** (everything else).
  **Quantico has only weights 400 and 700** — do not write 500 or 600.
- **Two accent tokens, and the split matters.** `--color-accent` `#b0d000` is
  for *fills only*, always carrying `--color-on-accent` text. `--color-accent-ink`
  `#647500` is the same hue dark enough to work as a 1px line, as text, or as a
  focus ring on a light panel — `#b0d000` measures roughly 1.4:1 there and
  vanishes. Picking the wrong one either glares or disappears.
- Both are *UI chrome*. `#ff5005` orange is *artwork*. Never let chrome and
  artwork share a token.

## Skills

### Always use skills

Before starting any task, check whether an installed skill covers it — and if one does, invoke it rather than working from general knowledge. This is the default, not an escalation path. Design, frontend, and visual work in this project are expected to go through the relevant skill.

Skills may be composed. A landing page build can reasonably run `design-brief` → `design-tokens` → `frontend-design` → `design-review`, or use `design-flow` to orchestrate the whole sequence.

Only skip a skill when the task genuinely falls outside every installed skill's scope, or when the user asks for something quick and throwaway.

### Local skills take priority over global

Skills in `.agents/skills/` (symlinked into `.claude/skills/`) are the ones this project has deliberately chosen. When a skill name exists both here and in the global/built-in set, **use the local one.**

Names that currently collide with built-in skills — always resolve these to the local copy:

- `frontend-design`
- `full-output-enforcement`

If a skill listing shows a directory-scoped variant, prefer the scoped one for work inside this repo.

### Installed skills

Managed via `npx skills add <repo>`; inventory tracked in `skills-lock.json`.

**Process / workflow**

| Skill | Use for |
| --- | --- |
| `design-flow` | Full design-to-build sequence; orchestrates the other skills in order |
| `design-brief` | Interactive interview → written brief for a feature or page |
| `brief-to-tasks` | Break a brief into ordered, independently buildable vertical slices |
| `grill-me` | Stress-test a plan or design through relentless questioning |
| `information-architecture` | Navigation, content hierarchy, URL patterns, user flows |
| `design-review` | Structured critique: hierarchy, consistency, responsiveness, a11y |

**Build / implementation**

| Skill | Use for |
| --- | --- |
| `frontend-design` | Production frontend built around a named aesthetic philosophy |
| `design-taste-frontend` | Anti-slop landing pages, portfolios, redesigns (v2, current default) |
| `design-taste-frontend-v1` | Original v1 behavior; only for exact backward compatibility |
| `design-tokens` | CSS variables / Tailwind config, light + dark, spacing and type scales |
| `high-end-visual-design` | Agency-grade fonts, spacing, shadows, cards, animation |
| `redesign-existing-projects` | Audit and upgrade existing UI without breaking functionality |
| `image-to-code` | Generate design images first, analyze, then implement to match |
| `gpt-taste` | Editorial typography + advanced GSAP scroll motion, AIDA structure |
| `stitch-design-taste` | Generate agent-facing `DESIGN.md` design-system specs |
| `full-output-enforcement` | Force complete, unabridged code output; bans placeholder stubs |

**Aesthetic directions**

| Skill | Use for |
| --- | --- |
| `minimalist-ui` | Warm monochrome, typographic contrast, flat bento, no gradients |
| `industrial-brutalist-ui` | Swiss print × military terminal; rigid grids, analog degradation |
| `brandkit` | Brand-guideline boards, logo systems, identity decks |
| `imagegen-frontend-web` | Web design references — one horizontal image per section |
| `imagegen-frontend-mobile` | Mobile screen concepts and flows in phone mockups; images only |

Skills run with full agent permissions. Review a skill's `SKILL.md` before first use.
