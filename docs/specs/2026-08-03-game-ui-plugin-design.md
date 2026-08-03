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

- Serve greenfield game UI (own visual identity) **and** UI for an existing game
  (indistinguishable from the host).
- Carry a game-specific constraint layer that is concrete enough to decide
  right from wrong, not a restatement of textbook principles.
- Stay engine-agnostic in the skills themselves so any new project qualifies.

## Non-goals

- No engine-specific skill body. Engine specifics live in appendices.
- No scripts or tooling in v1. This is a knowledge skill.
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
skill, whose success criterion ("a player takes it for vanilla") is the opposite
of its purpose.

Therefore both descriptions are written along the *form* axis. The overhaul then
falls cleanly on the "own form" side and the router hits it directly.

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

### D6 — Repository location

`claude-plugins`, one repo carrying both the
marketplace manifest and the plugin, via a relative `./plugins/<name>` source —
the single-repo pattern.

Remote: `backup` on a self-hosted Git server. No GitHub
`origin` for now; whether this becomes a public marketplace is an open question
(see below). The layout is identical either way, so the later move is
`git remote add origin …` plus `/plugin marketplace add` — no restructuring.

## Layout

```
claude-plugins/
├── .claude-plugin/marketplace.json     # "source": "./plugins/game-ui"
└── plugins/game-ui/
    ├── .claude-plugin/plugin.json      # name: game-ui, version: 0.1.0
    ├── skills/
    │   ├── game-ui-design/SKILL.md
    │   └── game-ui-modding/SKILL.md
    └── docs/
        ├── input-and-focus.md
        ├── readability.md
        ├── raster-and-scaling.md
        ├── localization.md
        └── engines/unity.md
```

Invocation names become `game-ui:game-ui-design` and `game-ui:game-ui-modding`.
The redundancy is deliberate — shorter names (`design`, `modding`) lose their
meaning when the file is seen in isolation.

The marketplace is named **`valgard-plugins`**, not `claude-plugins` after the
repo: marketplace names share one namespace with `claude-plugins-official`, and a
near-identical name there is a hazard. The plugin inside it is `game-ui`.

## The two descriptions

These are the load-bearing artifacts; the router sees nothing else.

```yaml
# skills/game-ui-design/SKILL.md
name: game-ui-design
description: Use when the UI should carry its OWN visual identity — a new game, or
  a mod/overhaul that deliberately replaces the host game's interface with a visual
  language of its own. Covers aesthetic direction, typography and layout for HUD,
  menus, inventory, dialogue and onboarding, plus the game-specific constraint
  layer (controller focus traversal, diegetic layering, readability under motion,
  safe areas, pixel raster, localization in fixed boxes). NOT for UI that must
  blend into an existing game's look — use game-ui-modding instead.
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

## Mode directive — the divergent part

Only this block differs between the skills.

| | `game-ui-design` | `game-ui-modding` |
|---|---|---|
| Authority | the game's fiction | the host game |
| Core operation | invent | **derive** |
| Success criterion | "unmistakable, could be no other game" | "a player takes it for vanilla" |
| Risk | one justifiable risk is required | any risk is a defect |
| Diegetic layer | a deliberate choice (diegetic / meta / spatial / non-diegetic) | given by the host; recognized and kept |
| Typography | pairing is designed | the font is a finding, not a choice — read it, don't pick it |
| Palette | 4–6 named hex values are set | **measured** from host assets, never estimated |

The failure modes are opposite, which is why one shared directive could not work:
greenfield fails as **generic** (reaching for a default look), modding fails as
**recognizably foreign** (the model "improves" the host — rounder corners, higher
contrast, a more modern panel — and the result reads as a mod instantly).

### Anti-default calibration for `game-ui-design`

The counterpart to the three web looks `frontend-design` names. AI-generated game
UI clusters into four:

1. **Sci-fi HUD** — semi-transparent dark panel, cyan glow outline, thin
   technical all-caps sans, corner brackets as decoration. By far the most common.
2. **Fantasy parchment** — beige texture, serif display, gold ornament frame,
   wax-seal buttons.
3. **Mobile flat** — white icons on 60% black, thick rounded buttons, saturated
   green/red for yes/no, everything centered.
4. **Indie retro pixel** — midnight-blue panel, 1px white outline, 8×8 bitmap
   font, hearts as a health meter.

All four are legitimate for some briefs. What marks them as defaults rather than
choices is that they appear regardless of the game.

## Shared mechanics layer (`docs/`)

Identical in application scope for both skills; only the diegetic layer differs
(a choice in greenfield, a given in modding).

**`input-and-focus.md`** — a controller is not a keyboard with different keys:
explicit focus neighbours (geometric auto-derivation fails on irregular grids);
the initial focus on open is a decision, never "nothing focused"; hover state and
focus state are two states, not one; selection outside the viewport pulls the
scroll along, computed pivot-correct; button prompts follow the active device;
nothing reachable by hover only.

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
against bleeding.

**`localization.md`** — German runs ~30% longer than English, so fixed boxes
break there first; **glyph coverage** (an atlas or bitmap font has a fixed
character set; one missing glyph silently swaps in a fallback face and breaks the
look in place); no string concatenation; placeholders must be reorderable; plural
rules are language-dependent; raw term keys in the UI are a registration fault,
not a design problem.

Three of these generalize findings from Core Keeper mod work: sprite distortion
at positions exactly `k/16` is an instance of texel snapping; UI dimming at equal
absolute Z is an instance of sort-by-Z; a font variant without umlauts is an
instance of glyph coverage. They go in as **generic rules with a concrete
example**, naming the game but never the private build environment — so the files
stay publishable.

**Every `docs/` file ends with a verification line:** how do you establish that
this rule is violated in *this* project? For texel snapping: move a sprite slowly
across the screen and watch for edge shimmer. This is the difference between a
skill that advises and one that prescribes checks.

## Process

The `frontend-design` sequence is kept; the first phase inverts.

| | `game-ui-design` | `game-ui-modding` |
|---|---|---|
| 1 | **Invent**: palette, type, layout, signature — plus diegetic layer and input model | **Measure**: palette by pixel sample, spacing grid from existing panels, font as a finding, border weight, recognize the diegetic layer |
| 2 | Self-review against the four default clusters | Fix the findings as a token table |
| 3 | Build to the plan | Build **against the table** |
| 4 | Screenshot critique: "is it unmistakable?" | Side by side with a vanilla screenshot: "does it stand out?" |

Carried over unchanged into both: restraint ("remove one accessory"), screenshot
self-critique, and the full UX-writing section — with one inversion, in that the
modding skill matches the **host's voice**, not its own. If the game says
"backpack", it is not "inventory".

## Verification gate (`game-ui-modding`)

A mandatory section, because without it "derive the look from the host" is a
statement of intent the model can satisfy with plausible-sounding numbers — an
unmeasured `#1a2a2e` looks exactly like a measured one in the token table.

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
- [ ] Palette sampled from a screenshot of the running game, not estimated
- [ ] Font confirmed available *including the special characters of every shipped
      language* — one missing glyph silently swaps in a fallback face
- [ ] Spacing and border weight measured on an existing host panel
- [ ] Interaction verified in-game: what closes it, what holds focus on open,
      which glyphs the button prompts show per input device
- [ ] Any decompiled finding traced through the whole chain, not a single link

### Decompiled sources, when available

A decompiled build tells you *why* a value is what it is; observation only tells
you *that* it is. Use both. But a decompiled finding counts only once the whole
chain has been read — never a single link:

    serialized asset / prefab value
      → constructor / init
      → runtime overrides (theme, settings, UI scale, localization)
      → renderer / shader / sorting
      → what actually reaches the screen

Reading one link produces confident wrong answers: a field that is authored but
dead, a value overwritten a frame later, a sort order the pipeline ignores, a
font that resolves to a fallback face only at runtime. Each of those looks
perfectly conclusive in isolation.

A decompiled source never replaces observation — it explains it. Where the chain
cannot be followed to the end, the finding is unconfirmed and measurement stands.

When an observation contradicts the table, the observation wins and the table
changes. Never the other way round.

Ship criterion: an A/B screenshot pair — host UI and yours, same scene, same
scale. If you can tell which one is the mod, it is not done.
```

The point of the last paragraph: the typical failure is not skipping the check,
but running it, seeing it contradict the assumption, and explaining the
difference away as measurement error.

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

A misrouted prompt is a description defect, not a user error.

## Scope

- Each SKILL.md 100–150 lines (house convention is 93–232; `frontend-design` is 56).
- Each `docs/` file 40–80 lines.
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
