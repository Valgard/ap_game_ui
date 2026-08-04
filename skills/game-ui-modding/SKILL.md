---
name: game-ui-modding
description: Use when the UI must be INDISTINGUISHABLE from an existing game's own
  interface — added panels, fields, menus or plugin UI that should read as part of
  the host game. Covers deriving the host's visual language by measurement instead
  of inventing one, plus the game-specific constraint layer (controller focus
  traversal, diegetic layering, readability under motion, safe areas, pixel
  raster, localization in fixed boxes). NOT for a deliberate overhaul that
  replaces the host's look — use game-ui-design instead.
---

# Game UI modding

The host game is the authority, not your taste. The goal is not "fitting in" —
it is indistinguishable: a player, or the host's own QA pass, cannot pick your
panel out of the rest of the interface. The core operation is **derive**, never
invent. Any risk you take on the host's behalf — rounder corners, higher
contrast, a "cleaner" panel — is a defect, not a flourish. The diegetic layer is
not yours to choose: recognize whichever one the host already uses and keep it.
Typography is a finding, not a choice — read the font that ships, don't pick one
you like better. The palette is measured from host assets, never estimated.
Success means the host's own designers would have built it this way.

## If the brief wants its own look

If the brief is a deliberate overhaul that replaces the host's own look with a
visual language of its own, this is the wrong skill; use `game-ui-design`
instead.

## Grammar and motif

"Indistinguishable" binds the **grammar**, not the motif. A mod may add a
subsystem with its own iconography, colour accent or diegetic conceit — a
faction console, an in-fiction terminal — exactly as a vanilla update
introduces a screen of its own. Binding: the spacing grid, the font, border
weight, panel construction, interaction and focus conventions, button-prompt
style. Free: iconography, colour accent, diegetic conceit.

**Original assets are not new grammar.** Whether a sprite was drawn for the mod
is irrelevant; what decides is whether it obeys the host's construction rules —
measured spacing, pixel grid, outline weight, palette range. A hand-drawn
9-slice frame that matches the host's inventory margin is motif; the same
frame two pixels thicker is grammar. Match the host's style, not its art. The
test is not "is anything new here" but **would the host's own designers have
built it this way**.

**Scope does not change the verdict.** New grammar for any subsystem — the
inventory alone, the map alone — is `game-ui-design` work, not modding at
reduced size; routing on proportion would put the most demanding mods in the
skill built to forbid what they need. What the partial case adds is an
obligation the full replacement does not have: **the seams**, where new
grammar meets the untouched rest — a panel excellent alone and jarring beside
its neighbour has failed at the only place a player sees both.

## Verify every assumption against the running game

Every statement about the host is a hypothesis until you have seen it. "The
panel background is `#1a2a2e`", "there is a bold weight", "Escape closes this
menu" — each sounds plausible, and a wrong guess here is exactly what makes a
mod read as foreign. The check is the running game — not the editor (camera,
post-processing, UI scale and font fallback all differ), not the wiki
(describes some version, not the installed one), not memory.

### Checklist before writing the token table

- [ ] Preconditions established by looking, not inferred — gamepad, typed
      input, a UI scale at all — each recorded applicable or `N/A — <reason>`
- [ ] Palette sampled from a lossless still, not estimated
- [ ] Font confirmed available including the special characters of every
      shipped language
- [ ] Spacing and border weight measured on an existing host panel
- [ ] Interaction verified in-game: what closes it, what holds focus on open,
      which glyphs the button prompts show per input device
- [ ] Any decompiled finding traced through the whole chain, not one link

Record each item using the schemas at
`${CLAUDE_PLUGIN_ROOT}/docs/verification-gate.md` — read it before writing the
first row of either table below.

### Two records

Both sit beside the implementation, and both are required: **token
provenance** — every value with how it was obtained, a `measured` row naming
its artifact, game version, and any setting that affects it; and **rule
applicability** — a precondition is not a token and does not fit that schema,
so it gets its own record. A precondition nobody wrote down cannot be checked
later.

### When the gate is passed, and when it is not

`ASSUMED` and `N/A` are legal entries; an unmarked value is not — a guess
formatted like a measurement is exactly the failure this gate exists to
prevent. What makes `ASSUMED` and `N/A` legal is the discipline around them:

1. **Completeness** — every token the implementation uses has a row. A value
   in the code with no row is a defect, not an omission.
2. **Identifiability** — every `measured` row names artifact, version and the
   settings that affect it, *and* the named artifact exists at the named path,
   produced by the capture commands `capturing-evidence.md` documents. A path
   nothing was ever saved to makes the row a guess wearing a citation.
3. **A reason, not a shrug** — `ASSUMED` states why verification was
   impossible and what would settle it. "Not checked" without a reason is
   incomplete, not `ASSUMED`.
4. **Saturation limit** — if the load-bearing tokens (palette, font, spacing)
   are *all* `ASSUMED`, the gate is **not passed**: the result is a draft, and
   it is reported as one. A gate satisfiable with nothing measured is
   decoration.

### When verification is not possible

A build may not run, a platform or input device may not be at hand,
decompilation may be unavailable. Then: mark every affected token `ASSUMED`
and say so in the answer, not only in the table; name what would settle it
("a screenshot of the crafting panel at UI scale 1.0 would confirm the border
weight"); never upgrade an `ASSUMED` value later without the observation that
earns it. Blocking on a missing build is wrong — the work proceeds. Presenting
assumptions as findings is the failure mode.

### Decompiled sources

A decompiled build explains why a value is what it is; it never replaces
seeing that it is. It counts only once the whole chain — authored data,
construction, runtime overrides, renderer, what reaches the screen — has been
read; one link produces confident wrong answers. Where the chain cannot be
followed to the end, the finding is unconfirmed and measurement stands. The
chain's five links and what each hides are read at
`${CLAUDE_PLUGIN_ROOT}/docs/verification-gate.md` before trusting any
decompiled finding.

### Observation wins

When an observation contradicts the table, the observation wins and the table
changes — never the other way round. The typical failure is not skipping the
check, but running it, seeing it contradict the assumption, and explaining the
difference away as measurement error.

## Ship criteria

A lossless still settles the A/B pair against the host (same scene, same
scale — if you can tell which is the mod, it is not done) and every
measurement; never sample a colour out of a recording, video is lossy. A
screen recording settles anything that exists only in time — focus traversal,
input capture, cancel chains, readability in motion — recorded against a
written sequence, since an unscripted clip proves only what it happened to
contain. Neither settles input latency or a state that was never triggered.
Consult `${CLAUDE_PLUGIN_ROOT}/docs/verification-gate.md` for the full
axis-to-artifact mapping, the sequence template, and the extraction cadences.
This is a review protocol, not a release certificate.

## Process

Measure the host → fix the findings as a token table with provenance → build
against the table, not from memory → A/B against vanilla and ask "does
anything stand out?"

## Constraints

The mechanics layer below decides right from wrong the same way in
`game-ui-design`; read each file for the reason given, before the work it
governs:

- Before wiring focus order, button prompts, or any field you add, read
  `${CLAUDE_PLUGIN_ROOT}/docs/input-and-focus.md`.
- Before sizing text you add or placing it near a screen edge, read
  `${CLAUDE_PLUGIN_ROOT}/docs/readability.md`.
- If any host art is pixel-based or point-filtered, read
  `${CLAUDE_PLUGIN_ROOT}/docs/raster-and-scaling.md` before positioning or
  scaling anything you add.
- Before locking a box size, a font, or any string shipping in more than one
  language, read `${CLAUDE_PLUGIN_ROOT}/docs/localization.md`.
- Consult `${CLAUDE_PLUGIN_ROOT}/docs/engines/unity.md` when the host runs on
  Unity, to establish which UI system it actually uses before any rule above
  is applied to it.
- `${CLAUDE_PLUGIN_ROOT}/docs/verification-gate.md` holds the schemas and
  templates already pointed at above; read its cadence rule before opening
  `capturing-evidence.md`, which is not linked here.

## Writing in the host's voice

Write in the host's vocabulary, not your own. If the game calls it a
"backpack", your added row is never "inventory"; don't rename a resource to
the genre default when the host already named it something else. Keep every
convention the host already set: active voice, a button's result keeps its
label ("Craft" logs "Crafted," never "Item created"), an error states what
happened and how to fix it without apologizing, an empty list reads as an
invitation the same way the host's own empty states do — match its tone, not
only its layout.
