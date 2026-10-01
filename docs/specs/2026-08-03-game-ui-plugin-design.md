# Design: `game-ui` plugin — UI/UX skills for games

Date: 2026-08-03

## Context

Anthropic ships a `frontend-design` skill (in the `claude-plugins-official`
marketplace) for "distinctive, intentional visual design when building new UI".
Superpowers references it by name as the implementation-layer counterpart to its
own process skills — `using-superpowers/SKILL.md` and
`brainstorming/SKILL.md:61` both name it explicitly.

`frontend-design` is usable for game UI, but only partly:

- Its **process** (plan a token system → self-review against the brief → only
  then write code) and its **UX-writing section** are medium-independent.
- Its **aesthetic directive** ("be unmistakable, take one justifiable risk") is
  correct for a new game and actively wrong for a mod that must blend in.
- Its **constraint layer is web-shaped**: CSS selector specificity, responsive
  breakpoints, scroll-triggered reveals, hover micro-interactions. The
  game-specific layer — controller focus traversal, diegetic layering,
  readability under motion, safe areas, pixel raster, localization in fixed
  boxes — is absent.

No game-UI counterpart exists. Of 276 plugins in the official marketplace,
exactly one has a game angle (`unreal-engine-skills-for-claude-code`), and it is
an MCP editor bridge, not a design skill.

## Goals

- Serve greenfield game UI (its own visual identity) **and** UI for an existing
  game (indistinguishable *in the host's grammar* — see the grammar-vs-motif rule,
  which is what "indistinguishable" means throughout this spec).
- Carry a game-specific constraint layer that is concrete enough to decide
  right from wrong, not a restatement of textbook principles.
- Stay engine-agnostic in the skills themselves so any new project qualifies.

## Non-goals

- No engine-specific skill body. Engine specifics live in appendices.
- No scripts or tooling in v1. This is a knowledge skill; documented one-liners for
  capture and frame extraction are not tooling.
- No replacement for `frontend-design` on web work.

## Decisions

### D1 — Two skills, not one

The aesthetic directive is genuinely opposed between the two cases: "be
unmistakable" vs. "be unnoticeable". A single skill would have to hold both in
one document and pick at runtime.

Splitting also moves mode selection **into the router**, where it is decided by
frontmatter `description` before the body is ever read — structurally more
reliable than a gate inside the text.

### D2 — Bundled as a plugin, not as loose skills

A plugin (`.claude-plugin/plugin.json` + `skills/<name>/` + plugin-wide `docs/`)
makes the shared mechanics layer a single source of truth without cost: the unit
of portability is the plugin, so both skills and the shared docs always travel
together. This is exactly how Superpowers is laid out (`skills/` alongside
`docs/`, `assets/`, `hooks/`, `scripts/`).

### D3 — The mode axis is "own form vs. host form", not "new vs. existing"

A UI **overhaul** mod (pattern: SkyUI) is a mod, but deliberately replaces the
host's look. Under a "new vs. existing" axis it would route to the modding
skill, whose success criterion — that the host's own designers could have built it
— is the opposite of its purpose.

Therefore both descriptions are written along the *form* axis. The overhaul then
falls cleanly on the "own form" side and the router hits it directly.

**A third case the axis catches and an earlier draft's descriptions did not.** A
studio maintaining the UI of its *own* shipped game — a HUD clipping on ultrawide,
broken gamepad focus in the options menu, a language being added — is own-form work:
the grammar is theirs to change if they choose to. It is not a new game, not a mod,
and not "blending into an existing game" because it *is* that game. The first
descriptions enumerated project types (new game / mod) and so covered none of it,
leaving the whole mechanics layer unreachable for maintenance and platform-porting
work. Fixed by naming the axis in the description — "the UI's form is yours to set" —
with the project types as examples rather than as the definition. The distinguishing
question is not how old the game is but **who may change the grammar**.

### D4 — No skill-to-skill handoff as the primary mechanism

There is no dependency or successor field in skill frontmatter; Anthropic's
`skill-development` documents `references/` for skill-internal files only.
Superpowers chains **by instruction in prose** — and pays for it: to make one
transition (`brainstorming` → `writing-plans`) reliable it uses a `<HARD-GATE>`
block, a graphviz terminal node, a checklist entry, "The ONLY skill you invoke",
and a repetition at the end of the file. Five redundancies for one hop.

A cross-reference stays in each skill body as a **safety net** (two lines: "if
the brief demands the opposite, the sibling skill applies"), but the routing
itself is carried by D3.

### D5 — Engine-agnostic core, engine appendices in `docs/`

Both SKILL.md files stay engine-free. Engine specifics go to
`docs/engines/<engine>.md`, read only on demand. Rationale: a skill's leverage
comes from checkable statements; pure principles would restate what the model
already has, and an engine-bound skill would break the greenfield premise.

**An engine appendix names mechanisms, never values.** The distinction is not
pedantic — it is the difference between a true statement and a false one. Unity has
a `spritePixelsToUnits` field on the sprite importer; that 16 is its value in a
given game is a fact about that game. Unity has `TransparencySortMode` on the
camera; which mode is set is a fact about that game. Unity offers uGUI/Canvas,
SpriteRenderer, and hand-rolled retained-mode UI; **which one a game uses is a
choice the game made**, and it is the first thing to establish.

Two Unity games, measured: Core Keeper 1.2.1.4 draws its UI as SpriteRenderer on a
GUI layer with its own element base — no Canvas. Cities: Skylines 1 uses
`ColossalFramework.UI` (`UIComponent`/`UIPanel`/`UIView`, 114 files in the
decompiled `ColossalManaged`) with **zero** references to `UnityEngine.UI` and
**zero** to `SpriteRenderer`. Same engine, disjoint architectures, and neither one
uses the framework Unity ships.

So an appendix says *which mechanisms exist and where to read their values in this
project*. A value copied from one game into the appendix is the label problem, not
the measurement problem: correctly measured, then filed under a heading that claims
a reach it never had.

### D6 — Repository location

**Superseded on 2026-10-01.** This decision originally put the marketplace manifest
and the plugin in one repo, `claude-plugins`, with a
relative `./plugins/<name>` source — the single-repo pattern. That held
while the marketplace carried exactly one plugin and both were edited together.

They are now two repositories, following the index-repo pattern
instead, where the marketplace is an **index** and plugins live in their own repos:

| | |
|---|---|
| plugin | repo `ap_game-ui` — `.claude-plugin/plugin.json` at the **root**, which is what `source: "url"` requires |
| marketplace | repo `agent-plugins` — catalog only, no plugin content |

What changed materially, beyond the paths:

- **The source form.** A relative path cannot cross a repository boundary, so the
  catalog entry became `{"source": "url", "url": "…/ap_game-ui.git", "ref": "<tag>"}`.
- **The plugin gained a release notion.** In the single repo there was no version to
  speak of: the catalog pointed at a path and whatever lay there was installed. The
  `ref` now pins a tag, so the catalog and the plugin can move independently.
- **The marketplace name stayed `valgard-plugins`** even though the repo is called
  `agent-plugins`, because `agent-plugins` is already taken by another catalog
  in this machine's marketplace namespace and `install <x>@agent-plugins` would be
  ambiguous.

Remote: `backup` on a self-hosted Git server. `/plugin marketplace add` rejects
`ssh://host:port/…` as an invalid source format and the SCP short form cannot encode
a port, so the HTTP clone URL is the only working remote form for a self-hosted Git host on a
non-standard SSH port — see *Open questions* for the still-undecided GitHub question.

## Layout

Two repositories since 2026-10-01 (see D6). The plugin repo:

```
ap_game-ui/
├── .claude-plugin/plugin.json          # name: game-ui, version: 0.1.0 — at the ROOT
├── skills/
│   ├── game-ui-design/SKILL.md
│   └── game-ui-modding/SKILL.md
└── docs/
    ├── input-and-focus.md
    ├── readability.md
    ├── raster-and-scaling.md
    ├── localization.md
    ├── verification-gate.md
    ├── capturing-evidence.md
    ├── engines/unity.md
    └── specs/2026-08-03-game-ui-plugin-design.md   # this document
```

The marketplace repo carries no plugin content — only the catalog that points here:

```
agent-plugins/
└── .claude-plugin/marketplace.json     # name: valgard-plugins
                                        # source: {url, ref} → ap_game-ui.git
```

`${CLAUDE_PLUGIN_ROOT}` resolves to the plugin repo's root, so every reference in the
skills kept working unchanged across the split — the paths were always relative to
that variable, never to the old monorepo.

Invocation names become `game-ui:game-ui-design` and `game-ui:game-ui-modding`.
The redundancy is deliberate — shorter names (`design`, `modding`) lose their
meaning when the file is seen in isolation.

The marketplace is named **`valgard-plugins`**, not `claude-plugins` after the
repo: marketplace names share one namespace with `claude-plugins-official`, and a
near-identical name there is a hazard. The plugin inside it is `game-ui`.

**How the skills reach the shared `docs/`.** Both SKILL.md files reference them as
`${CLAUDE_PLUGIN_ROOT}/docs/<file>.md` — the documented variable for portable paths
inside a plugin. A relative path would break as soon as the plugin is installed
somewhere else. Bundled resources are not loaded automatically, so each reference
also states *when* to read the file ("before writing the token table, read
`input-and-focus.md`"), not merely that it exists. A file nobody is told to open is
a file that stays closed.

## The two descriptions

These are the load-bearing artifacts; the router sees nothing else.

```yaml
# skills/game-ui-design/SKILL.md
name: game-ui-design
description: Use when the UI's form is yours to set — a new game, your own shipped
  game's interface (maintenance, a platform port, a new language), or a mod/overhaul
  that deliberately replaces a host game's interface with a visual language of its
  own. Also the default when a brief names a game and a UI surface without saying who
  owns the form — "HUD for a space game", "inventory screen", "settings menu". Covers
  aesthetic direction, typography and layout for HUD, menus, inventory, dialogue and
  onboarding, plus the game-specific constraint layer (controller focus traversal,
  diegetic layering, readability under motion, safe areas, pixel raster, localization
  in fixed boxes). NOT for UI that must blend into someone else's game — use
  game-ui-modding instead.
```

```yaml
# skills/game-ui-modding/SKILL.md
name: game-ui-modding
description: Use when the UI must be INDISTINGUISHABLE from an existing game's own
  interface — added panels, fields, menus or plugin UI that should read as part of
  the host game. Covers deriving the host's visual language by measurement instead
  of inventing one, plus the game-specific constraint layer (controller focus
  traversal, diegetic layering, readability under motion, safe areas, pixel
  raster, localization in fixed boxes). NOT for a deliberate overhaul that
  replaces the host's look — use game-ui-design instead.
```

Two properties are intentional:

1. **The `NOT for … — use X instead` clause.** Describing only one's own scope is
   not enough: both skills sound plausible for "build me an inventory UI". Naming
   the sibling makes the wrong pick expensive rather than merely unlikely.
   Anthropic uses the same device in `claude-api` ("SKIP only when another
   provider is being worked on").
2. **The constraint list is verbatim identical in both.** It signals that either
   skill brings the mechanics layer, so the choice hangs on the mode axis alone
   and not on which description happens to carry the matching keyword.

## What is shared and what is mode-specific

**Shared verbatim by both skills:** the mechanics layer (`docs/`), the process
skeleton, restraint and visual self-critique, and the UX-writing section.

**Mode-specific:**

| Element | `game-ui-design` | `game-ui-modding` |
|---|---|---|
| Mode directive (table below) | own form | host form |
| Verification gate | reduced *only* where no host exists: invented values need no provenance, host mechanics facts still do | full: the host is the reference, so measurement is the core operation |
| Host-constraint section | present, for the overhaul case | not needed — the whole skill is about the host |
| UX-writing voice | its own | the host's |

## Mode directive — the divergent part

| | `game-ui-design` | `game-ui-modding` |
|---|---|---|
| Authority | the game's fiction | the host game |
| Core operation | invent | **derive** |
| Success criterion | "unmistakable, could be no other game" | "the host's own designers would have built it this way" (grammar, not motif — see *New content inside the host's grammar*) |
| Risk | one justifiable risk is required | any risk is a defect |
| Diegetic layer | a deliberate choice (diegetic / meta / spatial / non-diegetic) | given by the host; recognized and kept |
| Typography | pairing is designed | the font is a finding, not a choice — read it, don't pick it |
| Palette | 4–6 named hex values are set | **measured** from host assets, never estimated |

The failure modes are opposite, which is why one shared directive could not work:
greenfield fails as **generic** (reaching for a default look), modding fails as
**recognizably foreign** (the model "improves" the host — rounder corners, higher
contrast, a more modern panel — and the result reads as a mod instantly).

### New content inside the host's grammar

"Indistinguishable" binds the **grammar**, not the motif. A mod may add a
subsystem with its own iconography, colour accent or diegetic conceit — a faction
console, an in-fiction terminal, a screen belonging to one expansion — exactly as
a vanilla game introduces a new interface of its own. What stays binding is the
grammar: spacing grid, font, border weight, panel construction, interaction and
focus conventions, button-prompt style.

**Original assets are not new grammar.** Whether a sprite was drawn for the mod is
irrelevant. What decides is whether it obeys the host's construction rules —
measured spacing, pixel grid, outline weight, palette range. A hand-drawn 9-slice
panel frame that matches the host's inventory margin is motif; the same frame two
pixels thicker is grammar. **Match the host's style, not its art.**

That distinction matters because construction elements read as grammar at first
glance: a border, a scrollbar, a row background. They are motif as long as they are
built to the host's measurements, and a mod that authors every pixel itself can
still be entirely host-grammar work.

The test is therefore not "is anything new here" but **"would the host's own
designers have built it this way"**. New motif inside the host's grammar passes.
New grammar does not.

**Scope does not change the verdict.** New grammar for *any* subsystem — the
inventory alone, the map alone — is `game-ui-design` work, not modding at reduced
size. It is the historically common shape of an overhaul: SkyUI replaced inventory
and map, not the whole interface. Routing on proportion would put the most demanding
mods in the skill that forbids what they are for.

What the partial case adds is an obligation the full-replacement case does not have:
**the seams**. Where the new grammar meets the untouched rest, the transition is the
quality marker — a panel that is excellent alone and jarring next to its neighbour
has failed at the only place a player sees both.

This closes the gap the binary would otherwise leave: a mod can be neither
literally indistinguishable nor a full replacement, and without these rules such
work would land on a success criterion it cannot satisfy.

### Anti-default calibration for `game-ui-design`

The counterpart to the three web looks `frontend-design` names. These are the
recurring shapes to recognize — an open list, not a taxonomy:

1. **Sci-fi HUD** — semi-transparent dark panel, cyan glow outline, thin
   technical all-caps sans, corner brackets as decoration.
2. **Fantasy parchment** — beige texture, serif display, gold ornament frame,
   wax-seal buttons.
3. **Mobile flat** — white icons on 60% black, thick rounded buttons, saturated
   green/red for yes/no, everything centered.
4. **Indie retro pixel** — midnight-blue panel, 1px white outline, 8×8 bitmap
   font, hearts as a health meter.
5. **Muted dark minimal** — desaturated dark cards, hairline dividers, restrained
   neutral sans, sparse monoline icons, thin progress bars. The contemporary
   "tasteful" default.

Every one of them is legitimate for some brief. What marks a look as a default rather
than a choice is that **nothing in it points at this subject** — the same treatment
would serve a farming sim and a survival horror equally well. That is a claim about
how the look was arrived at, not a measured claim about how often it occurs in game
UI at large.

**No frequency claim is made.** There is no corpus behind this list; it is a set
of shapes to recognize, not a ranking. Cluster 5 warrants the most suspicion
precisely because restraint reads as deliberation — it is the one most likely to
survive a self-review that the other four would fail.

**Keeping it current:** when a design passes self-review and still feels
templated, name the shape and add it to this list.

## Shared mechanics layer (`docs/`)

The same files serve both skills; only the diegetic layer differs (a choice in
greenfield, a given in modding).

### Not every constraint applies to every game

Each rule below carries a precondition. A rule whose precondition is absent is not
a *relaxed* rule — for that project it does not exist:

| Rule | Precondition |
|---|---|
| **every rule below** | **which UI system the game actually uses** — established first, because it decides what the other rows even mean. Two Unity games measured: one draws UI as SpriteRenderer on a GUI layer, the other via its own `ColossalFramework.UI` with zero `UnityEngine.UI` references. The engine does not tell you; the game does |
| controller focus traversal, device-specific button prompts | the game accepts a gamepad |
| text entry vs. gameplay input, on-screen keyboard | the UI takes typed input |
| safe areas, overscan, cutout | output to a TV, or a device with a cutout |
| integer scaling, texel snapping, atlas padding | pixel art, or any point-filtered sprite |
| UI-scale extremes | the game exposes a UI scale — **many do not** |
| ultrawide and 4:3 | a platform that ships those ratios |
| RTL mirroring, contextual shaping | an RTL language is shipped |
| CJK line breaking, IME | a CJK language is shipped |

**Applicability is itself an observation, not an assumption.** "This game has no UI
scale" is a statement about the game and falls under the same gate as a colour
value: open the options screen. Do not infer it from the genre, the engine, or the
fact that most games have one.

Record the outcome as `N/A — <reason>` in the **rule applicability table** — a
precondition is not a token and does not fit that schema, so it has its own, see
*Two records, not one*. The reason `ASSUMED` exists is the reason this does:
otherwise "not applicable" and "not checked" are afterwards indistinguishable, and
a skipped check reads as a passed one.

**`input-and-focus.md`** — a controller is not a keyboard with different keys:
explicit focus neighbours (geometric auto-derivation fails on irregular grids);
the initial focus on open is a decision, never "nothing focused"; hover state and
focus state are two states, not one; selection outside the viewport pulls the
scroll along, computed pivot-correct; button prompts follow the active device;
nothing reachable by hover only.

**Focus has a lifecycle, not only a position** — and this spec's own example, a
filter field above a scrolling list, breaks every one of these: focus returns to its
origin when a nested panel or modal closes; when the focused row disappears because
it was filtered out, removed, disabled or reordered, focus moves by a stated rule
instead of vanishing; disabled and hidden controls are skipped consistently in every
direction, not only downward; and switching input device mid-focus keeps the
selection while swapping the prompts. Filtering *is* removing the focused row, so
this is the main interaction of that example rather than an edge case.

The same file's second half is **text entry and input capture**, because the
modding description explicitly covers added *fields*: while a field holds focus,
gameplay input is suppressed — otherwise WASD walks the character while the player
types; cancel priority is defined per nesting level (does Escape clear the field,
close the panel, or open the pause menu?); IME and composition must work or CJK
entry is impossible; a controller-only or handheld context needs an on-screen
keyboard path; and every new binding is checked against the host's existing
bindings before it is claimed.

**`readability.md`** — comprehension budget as a design quantity (health must
read in under a second, a crafting menu need not); text over a moving background
needs a carrier surface (outline, shadow, panel), contrast ratio alone is not
enough; safe areas (TV overscan, notch); HUD occlusion (corners are the most
expensive real estate, the center is off limits); colour is never the sole
carrier of information.

**`raster-and-scaling.md`** — the class of bug that passes as a "graphics
glitch": integer scaling and nearest-neighbour for pixel art; **texel snapping**
(round positions to texel boundaries or get distortion and shimmer in motion);
**sorting** (a pipeline that sorts by Z rather than by layer makes equal Z values
collide — visible as a colour or brightness fault, not as an ordering fault);
reference resolution plus UI scale instead of absolute pixel sizes; atlas padding
against bleeding. Reference resolution alone does not prevent clipping or an
unusable composition, so the layout is checked at the ratios and scales the target
platform actually ships: ultrawide 21:9 and 4:3 alongside 16:9, dynamic
resolution, OS display scaling, and — *where the game exposes one at all* — both
ends of its UI scale. Point-filtered art is not the only thing display scaling
affects: text and vector rasterization at fractional DPI blurs or shifts on its own
terms, so it is checked separately rather than assumed covered by the pixel rules.

**`localization.md`** — German runs roughly 30% longer than English for short UI
strings; that is a useful order of magnitude, not a ranking — Finnish and Russian
regularly run longer, and CJK fails on entirely different grounds. Which language is
longest is a property of the individual string, which is why the check covers every
shipped language instead of one presumed worst case. Further: **glyph coverage** (an atlas or bitmap font has a fixed
character set; one missing glyph silently swaps in a fallback face and breaks the
look in place); no string concatenation; placeholders must be reorderable; plural
rules are language-dependent; raw term keys in the UI are a registration fault,
not a design problem. Beyond Latin scripts: right-to-left languages mirror the
**layout**, not merely the text — focus order, icon direction and progress fill
all invert; Arabic needs contextual shaping, and what breaks it is
character-by-character rendering *without* a shaping step — an atlas is compatible
as long as it holds the shaped forms and selection happens after shaping; CJK breaks
lines per character rather than per word, invalidating word-wrap assumptions; and
strings of mixed direction need their own test case.

Three of the rules above generalize findings from the author's own Core Keeper mod
work on game version 1.2.1.4, each established in-game rather than inferred: sprite
distortion at positions exactly `k/16` is an instance of texel snapping; UI dimming
at equal absolute Z is an instance of sort-by-Z; a font variant without umlauts is an
instance of glyph coverage. They go in as **generic rules with a concrete example**,
naming the game and version so a reader can place them, but never the private build
environment — so the files stay publishable. The examples illustrate the rules; they
are not claims about any project the skill is later applied to, and they do not
exempt that project from establishing its own facts.

**`verification-gate.md`** — the gate's lookup half: the two table schemas with
worked rows, the five links of the decompilation chain and what each one hides, the
full axis-to-artifact mapping, the recording sequence template, and the extraction
cadences. Not the rules themselves — those stay in the skill, see *Verification
gate*. This file is what one consults with a concrete case in hand.

**`capturing-evidence.md`** — how the evidence is produced, listed per platform,
because the commands are OS-specific and must not leak into an OS-agnostic skill
body (the same mistake as an engine term would be): lossless stills, screen
recording with a time limit and visible clicks where the platform offers them, and
frame extraction for machine review. On this machine that is
`screencapture -v -V<sec> -k`, plus **two** extraction cadences because one rate
cannot serve both axes: `ffmpeg -i clip.mov -vf fps=4 seq-%03d.png` for sequence and
coverage, and `ffmpeg -i clip.mov -ss <t> -t 1 shimmer-%03d.png` at native rate over
a one-second window for per-frame faults. Other platforms get their own entry.

**Each of the four mechanics files and `engines/unity.md` ends with a verification
line** that names both the check and its artifact type: how do you establish that
this rule is violated in *this* project, and is that a still or a recording? For
texel snapping: move a sprite slowly across the screen and watch for edge shimmer —
a recording, because the fault does not exist in any single frame. This is the
difference between a skill that advises and one that prescribes checks.

`verification-gate.md` and `capturing-evidence.md` carry no verification line of
their own. They *are* the verification reference; a check for how to check the
checking instructions would close a circle rather than open one.

## Process

The `frontend-design` sequence is kept; the first phase inverts.

| | `game-ui-design` | `game-ui-modding` |
|---|---|---|
| 1 | **Invent**: palette, type, layout, signature — plus diegetic layer and input model | **Measure**: palette by pixel sample, spacing grid from existing panels, font as a finding, border weight, recognize the diegetic layer |
| 2 | Self-review against the default clusters | Fix the findings as a token table, each value tagged with its provenance |
| 3 | Build to the plan | Build **against the table** |
| 4 | Visual critique: "is it unmistakable?" | A/B against vanilla: "does it stand out?" |

Phase 4 is not one artifact in either mode: the still settles the visual axis, and
the time-based axes need a recording. See *Ship criteria, by evidence type*.

Carried over unchanged into both: restraint ("remove one accessory"), visual
self-critique, and the full UX-writing section — with one inversion, in that the
modding skill matches the **host's voice**, not its own. If the game says
"backpack", it is not "inventory".

## Verification gate

Full form in `game-ui-modding`, quoted below; the reduced variant for
`game-ui-design` follows after it.

The gate is mandatory because without it "derive the look from the host" is a
statement of intent the model can satisfy with plausible-sounding numbers — an
unmeasured `#1a2a2e` looks exactly like a measured one in the token table.

**What stays in the skill, and what is looked up.** The cut is not detail versus
rule — it is *does this change behaviour while being read*. The exit criteria change
what counts as finished, so they are inline. Table schemas, the decompilation chain
and the extraction cadences are consulted once a case arises, so they live in
`docs/verification-gate.md`. Outsourcing the exit criteria would repeat D4's
mistake: a gate binds only if it is read, and a reference is weaker than inline
text.

```markdown
## Verify every assumption against the running game

Every statement about the host game is a hypothesis until you have seen it.
"The panel background is #1a2a2e", "there is a bold weight", "Escape closes this
menu" — each has a plausible ring to it, and a wrong guess here is precisely what
makes a mod read as foreign.

The check is the running game — not the editor, not the wiki, not memory:
- Editor rendering is not game rendering: camera, post-processing, UI scale and
  font fallback all differ.
- A wiki or changelog describes some version, not the installed one.

Confirm by observation before writing the token table:
- [ ] Preconditions established by looking, not inferred — does the game take a
      gamepad, does the UI take typed input, is there a UI scale at all? Each one
      recorded as applicable or `N/A — <reason>`
- [ ] Palette sampled from a lossless still of the running game, not estimated
- [ ] Font confirmed available *including the special characters of every shipped
      language* — one missing glyph silently swaps in a fallback face
- [ ] Spacing and border weight measured on an existing host panel
- [ ] Interaction verified in-game: what closes it, what holds focus on open,
      which glyphs the button prompts show per input device
- [ ] Any decompiled finding traced through the whole chain, not a single link

### Two records, not one

Both sit beside the implementation, and both are required:

- **Token provenance** — every value with how it was obtained. A `measured` row names
  its artifact, the game version, and any setting that affects the value; "measured"
  on its own is a word, not a provenance.
- **Rule applicability** — a precondition is not a token and does not fit that
  schema, so it gets its own record. Greenfield keeps this one too: it *decides* its
  preconditions instead of discovering them, and a decision nobody wrote down cannot
  be looked up later.

Schemas and worked examples: `${CLAUDE_PLUGIN_ROOT}/docs/verification-gate.md`.

### When the gate is passed, and when it is not

`ASSUMED` and `N/A` are legal entries. What makes them legal is the discipline
around them:

1. **Completeness** — every token the implementation uses has a row. A value in the
   code with no row is a defect, not an omission.
2. **Identifiability** — every `measured` row names artifact, version and the
   settings that affect it.
3. **A reason, not a shrug** — `ASSUMED` states why verification was impossible and
   what would settle it. "Not checked" without a reason is incomplete, not
   `ASSUMED`.
4. **Saturation limit** — if the load-bearing tokens (palette, font, spacing) are
   *all* `ASSUMED`, the gate is **not** passed. The result is a draft and is
   reported as one. A gate that can be satisfied with nothing measured is
   decoration.

An unmarked value stays the one thing that is never legal: a guess formatted like a
measurement is exactly the failure this gate exists to prevent.

### When verification is not possible

A build may not run, the platform or input device may not be at hand,
decompilation may be unavailable or not permitted. Then:

1. Mark every affected token `ASSUMED`, and say so in the answer — not only in the
   table.
2. Name what would settle it: "a screenshot of the crafting panel at UI scale 1.0
   would confirm the border weight".
3. Never upgrade an `ASSUMED` value later without the observation that earns it.

Blocking on a missing build is wrong — the work proceeds. Presenting assumptions
as findings is the failure mode.

### Decompiled sources, when available

A decompiled build tells you *why* a value is what it is; observation tells you
*that* it is. Use both — but a decompiled finding counts only once the **whole
chain**, from authored data to what reaches the screen, has been read. One link
produces confident wrong answers: a field that is authored but dead, a value
overwritten a frame later, a sort order the pipeline ignores, a font that resolves to
a fallback only at runtime. Each looks perfectly conclusive in isolation.

A decompiled source never replaces observation — it explains it. Where the chain
cannot be followed to the end, the finding is unconfirmed and measurement stands.

The chain's five links and what each one hides:
`${CLAUDE_PLUGIN_ROOT}/docs/verification-gate.md`.

When an observation contradicts the table, the observation wins and the table
changes. Never the other way round.

### Ship criteria, by evidence type

For every axis that evidence *can* settle, exactly one artifact settles it — and two
axes are settled by neither.

- **Lossless still** — the A/B pair against the host (same scene, same scale; if you
  can tell which is the mod, it is not done) and *every* measurement. **Never sample a
  colour out of a recording:** video is lossy, so the number it yields looks like a
  measurement while being a guess with extra steps.
- **Screen recording** — anything that exists only in time, where a still is not
  merely weaker but structurally unable to show it: focus traversal, input capture
  while typing, cancel chains, readability in motion, texel shimmer. Recorded against
  a **written sequence** — an unscripted clip proves only what it happened to
  contain, and keystrokes are invisible unless the platform draws them.
- **Neither** — input latency, and states that were never triggered. Coverage is a
  property of the sequence, not of the medium.

The full axis-to-artifact mapping, the sequence template, and the extraction cadences
(a coarse rate for sequence, native rate for per-frame faults):
`${CLAUDE_PLUGIN_ROOT}/docs/verification-gate.md`.

None of this is blind judgement: it is a review protocol, not a release
certificate.
```

The point of the "observation wins" rule: the typical failure is not skipping the
check, but running it, seeing it contradict the assumption, and explaining the
difference away as measurement error.

**The reduced variant in `game-ui-design`.** Provenance attaches to where a value
came from, not to which skill is running:

- **Invented values** — palette, type, layout, signature — carry no provenance. They
  are decisions, and a decision has no source to cite.
- **Host mechanics facts** carry full provenance whenever a host exists at all, and
  it does in the overhaul case that D3 routes here. Font availability, input system,
  resolution model and the fiction's frame are statements about someone else's game;
  the gate treats them exactly as the modding skill would.

Greenfield with no host has only the first kind — which is why the gate is *reduced*
there, not *absent*.

Either way the mechanics layer needs its in-game checks, subject to the same
preconditions: focus traversal on each supported input device, one still per shipped
language (the longest string is an *additional* wrap test, never a substitute for
coverage), a sprite in motion at native frame rate, the UI scale at both ends where
there is one. Greenfield replaces "where did this value come from" with "does my own
decision survive the constraints" — and it decides its preconditions instead of
discovering them, which is exactly why the applicability table is not optional there:
nobody can look up later whether the game was meant to support a gamepad.

## `game-ui-design`: when the target is an existing game

A short section covering the overhaul case that D3 routes here. The aesthetics
are free, but the host's mechanics remain binding: font availability, input
system, resolution model, and the fiction's frame. This is a constraint, not a
handoff — it applies even when the skill is invoked directly.

## Routing verification

Empirical, not by self-assessment: load the plugin in a fresh session via
`claude --plugin-dir <path>` and run a fixed prompt list.

| Prompt | Expected |
|---|---|
| "Inventory UI for my new roguelike" | `game-ui-design` |
| "Add a filter field to the crafting menu in my Core Keeper mod" | `game-ui-modding` |
| "My mod's menu looks foreign next to the original" | `game-ui-modding` |
| "HUD for a space game" | `game-ui-design` |
| "Full UI overhaul for an existing RPG, own visual language" | `game-ui-design` |
| "Match my settings panel to the game's own panels" | `game-ui-modding` |
| "A window of my own, with my own sprites, that fits into the game's UI" | `game-ui-modding` |
| "Replace the inventory's layout and look, leave the rest of the game's UI alone" | `game-ui-design` |
| "Our own shipped game's HUD clips on ultrawide — fix it" | `game-ui-design` |

A misrouted prompt is a description defect, not a user error.

## Scope

- The two skills are **not** the same size. `game-ui-design` 120–**170** lines;
  `game-ui-modding` 170–200 — the gate's binding half (rule, checklist, exit
  criteria) is ~60 lines and stays inline; its lookup half moved to
  `docs/verification-gate.md`. Both sit inside the observed house range (93–232).
- **Line counts measure wrapping, not content, and are a weak budget.** Normalised to
  the 76-column wrap the mechanics files use, `verification-gate.md` is ~73 lines
  against its "~50" target while two files reported as *at* their ceiling merely wrap
  tightest. Treat the numbers as a guard against sprawl, not as a measurement: a
  needed rule is never cut to defend one. The design ceiling moved twice for exactly
  that reason: 150 → 165 so the four exit criteria could go inline, then 165 → 170 when
  a measured routing defect needed two more description lines. Both times the number
  was the only obstacle, which is the tell that the number was the wrong constraint.
- `docs/` files 60–120 lines. Not uniform: `input-and-focus.md` carries two halves
  (controller focus *and* text entry / input capture) and will sit at the top of
  that range, `readability.md` at the bottom. Two exceptions, both reference rather
  than reasoning: `verification-gate.md` at ~50 and `capturing-evidence.md` at ~40.
- No scripts in v1. A palette extractor ("screenshot in, dominant colours out")
  fits the measurement logic and can be added later, but is scope creep while the
  skills themselves do not exist.
- English throughout, per the documentation-language rule.

## Open questions

- **Public marketplace.** Deferred deliberately. The repo carries only the
  `backup` remote until decided; layout is already GitHub-ready.
- **Further engine appendices.** Only `docs/engines/unity.md` is planned. Godot
  or others get added when a project needs them.
- **ADR.** The routing decisions (D1, D3, D4) are the kind that are not
  reconstructible later and easy to undo by accident. Candidates for an ADR
  distilled from this spec once the plugin is implemented.
- **Own game, host grammar.** `game-ui-design`'s description now says "not for UI that
  must blend into *someone else's* game", but `game-ui-modding` still says "an existing
  game's own interface" without excluding your own. So "add a panel to our own game,
  matching our own existing look" is ambiguous: the form is yours to set (design) yet
  the brief asks for grammar conformance (modding). The honest answer is that modding's
  *method* — measure the existing grammar rather than invent one — is what that brief
  wants, whoever owns the game. Left open rather than patched blind: it wants a routing
  test, not a guess, and the eight-prompt list would need a ninth case for it.
