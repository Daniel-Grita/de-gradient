# De-Gradient — Review Gate (2026-09-02)

Re-run of the four Phase-7 lenses against the current `index.html` after Batches 1–8.
Compares each against the score at the last audit (2026-09-01) and calls out anything
still open.

Screenshots this pass:
`gate-desktop`, `gate-tablet`, `gate-mobile`, plus the Batch-8 stateful shots
`B-b8-dither-empty`, `B-b8-gdots-full`, `B-b8-dissolve`, and the offline-fonts run
`B-offline-fonts`.

Verified once more, static: file 158 KB, zero console exceptions on boot at desktop /
tablet / mobile, JS parses clean, CSS balanced, all `var()` references resolved.

---

## 1. UX Heuristics (Krug + Nielsen)

**Was:** the ux-heuristics audit (2026-09-01) drove the whole task list.
**Now: 10/10.**

### Quick Diagnostic (all pass)

| Row | Was | Now |
|---|---|---|
| Can I tell what site/page this is? | ✓ | ✓ (`<h1>DE-GRADIENT</h1>`, wordmark) |
| Is the main action obvious? | ✓ | ✓ (Loop is primary green when applicable; Still when it isn't) |
| Is the navigation clear? | ✓ | ✓ (Generative / From image mode tabs + effect select) |
| Can I find the search? | n/a | n/a |
| Does the system show what's happening? | ✗ Still silent | ✓ status primitive covers Still success/error, Loop success/error, WebGL, image upload |
| Are error messages helpful? | ✗ state problem and stop | ✓ every error message carries a next step |
| Can users undo or go back? | ✗ preset apply destructive | ✓ dirty-guard + Revert + preset delete confirm; effect switch already preserved per-effect state |
| Does it work without hover? | ✗ hex fields invisible | ✓ hex + value inputs have persistent baseline underlines; ratio purpose is the one remaining hover-only affordance (title tooltip), acceptable per the brief |
| Are all interactive elements labeled? | ✓ | ✓ (aria-labels added on canvas + inputs; toggle label wired via for=id + label click handler) |
| Does anything make me stop and think "huh?" | ✗ Gradient colors collision, dot vs pixel vs cell, dimmed with no reason | ✓ collision resolved, naming unified, dim reasons in `title` |

### Nielsen 10 — where the improvements landed

- **Visibility of system status** (#1): status primitive with three kinds (info/success/error), REC badge, progress bar, meta line that says both PNG and MP4 dimensions per ratio.
- **Match real world** (#2): "Cell size" / "Pixel size" used consistently, "From gradient" / "Ink & paper" as the colour source, mode labels ("Generative" / "From image").
- **User control and freedom** (#3): dirty-guard on preset switch, Revert bypasses it (Revert *is* the discard), preset delete confirm, cancel button during recording.
- **Consistency** (#4): "Gradient colors" collision resolved; naming unified per concept.
- **Error prevention** (#5): Still disabled without image; save-preset blocks built-in name collisions; overwrite confirm on user preset collision; dirty guard on preset apply; WebGL-dependent effect options disabled with visible reason.
- **Recognition over recall** (#6): section hints (Noise, Dissolve, Edge, Motion, Tone), dim reasons on hover, tabular numerals for aligned values.
- **Flexibility** (#7): spacebar play/pause, Enter to commit typed value, Delete/Backspace to remove a user preset, Escape to cancel typed value.
- **Aesthetic minimalist** (#8): the two-level nav removed the always-visible second effect group; the panel is quieter than it was.
- **Recognize/diagnose/recover from errors** (#9): every error message includes a fix or next action. WebGL failure states what still works.
- **Help + documentation** (#10): contextual — section hints and tooltips carry the load; nothing else is needed for a single-file personal tool.

### Remaining, deliberately

- **Ratio purpose is hover-only** (`title` attributes: "square / video / portrait / vertical video"). The tight 4-segment row has no space for inline captions; on a desktop tool this trade is fine.
- **Touch-target pass not run.** Brief is explicit: desktop-first, phone tuning not a use case.

---

## 2. Design Review (against `PHASE7_REVIEW.md` must-fix list)

### Must-fixes — all closed

- ✅ **Heading hierarchy + landmarks.** One `<h1>` (DE-GRADIENT), seven `<h2>`s (section legends). `<main aria-label="Preview">` wraps the canvas area; panel stays `<aside>`.
- ✅ **`<canvas>` accessible name.** `#artboard` gets `role="img"` + a live `aria-label` that `syncStage()` updates to the active effect or "Drop an image to dither/convert" while empty.
- ✅ **Contrast.** `--color-legend-muted` #6e6e6e → #666666 (clears AA on both surface + raised); `.switch__seg` idle switched from muted → `--color-text` (8.6:1); `.is-inert{opacity:.4}` replaced with per-element dim tokens.

### Should-fixes — all closed (except touch targets, deferred)

- ✅ Focus ring quiet-but-legible; `:focus-visible` on hex + input so the ring shows only on keyboard focus.
- ✅ Dead tokens `--radius-lg` and `--space-1` removed.
- ✅ Aesthetic-fidelity notes ("active preset lime fills too loud", "+Save alone on last row") remain design decisions, not defects; unchanged.

### Could-improves that landed anyway

- ✅ Editable numeric values (was Sev-2, shipped).
- ✅ Per-section hint text (Sev-2, shipped).
- ✅ Persistent hex-field affordance (Sev-2, shipped).

### Fresh design-review pass (this build)

**Aesthetic fidelity:** faithful to the brief. The two-level nav reads as one instrument;
the effect select feels like the panel's title element. Nothing new is loud in the chrome.

**Responsive:** `gate-tablet` and `gate-mobile` both hold — panel drops below the frame,
frame keeps its ratio, hex fields and value inputs still show their persistent underlines.

**One aesthetic observation.** The persistent hex-field underlines can read a hair
too strong on the swatch rows at desktop; look at `gate-desktop` — the four
`#FF5005 / #DBBA95 / #D0BCE1` rules are more visible than the labels themselves. Not a
must-fix (they're now correctly-affordant, which was the point), but if it grates,
weaken the resting underline to `--color-surface-raised` and keep hover as `--color-seam`.

---

## 3. Norman — Design of Everyday Things

**Was:** 4/10 — two open gulfs (evaluation, recovery), one open constraint gap.
**Now: 10/10.** All five diagnostic rows pass.

| Row | Was | Now |
|---|---|---|
| Discoverability | ~ | ✓ section hints + tooltips + dim reasons + persistent hex/value underlines |
| **Evaluation** | ✗ Still silent, capture silent, WebGL silent | ✓ Every output action speaks (Exporting…/Saved X, or an actionable error) |
| **Error recovery** | ✗ no confirms, silent overwrite, no delete | ✓ dirty-guard, overwrite confirm, delete affordance (× + keyboard) |
| Mapping | ✓ | ✓ |
| Constraints | ✗ dead Still on empty | ✓ Still + Loop disabled together when the state can't produce the output; meta says why |

**Slip / mistake prevention** — the Norman model:

- **Slip:** typing into a hex or value field and hitting Enter now commits atomically; Escape restores. No accidental partial-value commits mid-typing.
- **Mistake:** picking a WebGL-dependent effect when WebGL is off is prevented at the source — those `<option>`s are disabled and their labels say "— needs WebGL".

---

## 4. Steve Jobs — Design Review

**Was:** NOT DONE (6/10). Passed the One Thing, ≤3 steps to value, cold review, real demo.
Failed: nothing was cut, back-of-fence was plywood, offline promise broken.

**Now: INSANELY GREAT (9/10).** Six of seven rows pass; one row deliberately stays out of scope.

```
# Design Review: De-Gradient
Verdict: INSANELY GREAT (score 9/10)
The One Thing: Open a file, tune a generative visual, take out a PNG or a seamless loop
               — no code, no account, no build step.
Keeps its promise?  Yes. The offline-genuine bit that made the previous review "not done"
                    is closed — 158 KB single file, three embedded fonts, zero external
                    requests verified with the network killed.
```

Diagnostic — before / after:

| Row | Was | Now |
|---|---|---|
| One Thing sayable in one sentence | ✓ | ✓ |
| Core value in ≤3 steps | ✓ | ✓ |
| Reviewed cold on a real running build | ✓ | ✓ |
| Working demo on real device | ✓ | ✓ |
| **Something removed this cycle?** | ✗ 20 additions, zero cuts | ✓ dropped the duplicate "add an image" affordance; `.switch__group` / `.switch__glabel` CSS deleted; `--radius-lg`, `--space-1` deleted; 5 CDN link tags deleted; both fonts embedded so external network is deleted |
| **Empty/error/edge states match hero quality?** | ✗ plywood failures | ✓ status primitive, WebGL sentence with what still works, error copy carries next steps, disabled Still + Loop label the reason in the meta line |
| Would the team sign it and use daily? | Partly | Effectively yes. **Daniel still hasn't opened it by eye** — this is the one row that's still on you, not the code. |

**The cut list left open on purpose** — waiting on Daniel's judgement:

- 5 charsets → 3 (Minimal + Dots feel like duplicates of Classic).
- Fold Wave scale into presets (a non-coder never touches it).

**The one back-of-fence I didn't audit this pass:** the loop MP4 file itself in real
Chrome. Bundled Chromium can't produce one to inspect. The zero-data path now surfaces
"Recording produced no data" instead of silence, but "the recorded loop plays back at
1920×1080 as a seamless MP4" is still a real-Chrome check.

---

## Consolidated — what closed, what's open

**Closed this session (all four scores now ≥9):**

- Status primitive + every silent path routes through it.
- Recovery gulf: dirty guard, preset delete, overwrite confirm, built-in name collision block.
- WebGL dead-end: `<option disabled>`, boot lands on Dissolve (a working generative effect),
  persistent boot status listing what still works.
- Variant B panel nav shipped (mode tabs + per-mode select), per-mode last-effect memory.
- Heading hierarchy, `<main>` landmark, `<canvas>` accessible name.
- Contrast pass (both muted and inert).
- WIG batch: `color-scheme:light`, `<meta theme-color>`, preconnect for Fontshare, `touch-action`,
  `overscroll-behavior:contain`, `:focus-visible` swaps, `type=url` + `inputmode` + `autocomplete=off`
  on the import field, curly apostrophes, explicit `<img>` dims, `translate="no"`, dead tokens gone.
- Offline promise closed: 158 KB single file, three fonts embedded, zero external requests
  verified with `Network.emulateNetworkConditions({offline:true})`.
- Loop output size: fixed per-ratio 1080p equivalents in RATIOS[i].lw/lh; buffer swap +
  restore around capture; window resize during recording no longer clobbers the buffer.
- Editable numeric values with parse / clamp / snap / suffix / Enter / Escape.
- Per-section hint text (data-driven from `EFFECTS`).
- Reason on dimmed controls (`activeWhen.reason`, surfaced as `title`).
- Ratio purpose (`RATIOS[i].purpose` in `title`).
- Naming consistency (Cell size / Pixel size).
- Gradient colors collision resolved.
- Persistent hex + value underlines.
- Spacebar play/pause.
- Toggle label + hover.
- loopTime as readout.
- Large-image guard.
- Cut: duplicate "add an image" button.

**Still open — needs Daniel:**

1. **Open it and look.** Every audit here has been on the running code; Daniel has not
   yet used the current build. This is the last row on the Jobs diagnostic.
2. **Cut list:** 5 charsets → 3? Fold Wave scale into presets?
3. **Real-Chrome MP4 run:** capture a full loop, confirm playback is seamless at 1920×1080.

**Intentionally deferred, documented in TASKS.md:**

- Touch-target pass (desktop-first per brief).
- Stabilise the primary button (positions already fixed; primary-colour follows applicability
  by design).
- Optional description line under the effect select (variant-B nice-to-have; needs a call).

**Ready to ship** — subject to Daniel opening it and confirming by eye.
