# Verification gate — reference

The rules live in the skill. This file is the lookup half: schemas, the chain, the mapping, and the templates. The exit criteria themselves — including the saturation limit — are inline in the skill, not here.

## Token provenance — schema

The rows below are an illustration of **form** — the shape a provenance entry takes. They are not measurements anyone took: `<Game> <version>` and every path are placeholders, and nothing here is a finding about a real game. Copy the shape, never the values.

| token | value | provenance |
|---|---|---|
| `panel.bg` | `#RRGGBB` read off the still | measured — `evidence/<panel>.png`, `<Game> <version>` |
| `border.w` | `<n>` px | measured — same still, top edge |
| `font.body` | host default | decompiled — chain traced to renderer, `<Game> <version>` |
| `corner.r` | 0 | ASSUMED — corners never sit against a contrasting background |

A measured row names the artifact, the game version, and any setting that affects the value; a bare "measured" is a word, not a provenance.

## Rule applicability — schema

| rule | applicable | evidence |
|---|---|---|
| controller focus traversal | yes | options screen lists gamepad bindings |
| UI-scale extremes | N/A | no UI scale in the options screen |
| RTL mirroring | N/A | no RTL language shipped |

## The decompilation chain — five links

    authored data (whatever the engine serializes it as)
      → construction / initialization
      → runtime overrides (theme, settings, UI scale, localization)
      → renderer / shader / sorting
      → what actually reaches the screen

Each link hides its own failure: a field authored but dead by construction, a value overwritten a frame after init, a runtime override that never touches the field you measured, a sort order the renderer ignores, a font resolving to a fallback only once it reaches the screen. A decompiled finding counts only once the chain has been read end to end.

## Axis to artifact — full mapping

| axis | artifact |
|---|---|
| palette, spacing, border weight | lossless still |
| font fallback | one still per shipped language |
| aspect ratios, UI-scale extremes | one still per configuration |
| A/B against the host | still pair, same scene and scale |
| focus traversal, input capture, cancel chains | recording |
| readability in motion, texel shimmer | recording, native frame rate |
| input latency | neither — high-frame-rate capture or instrumentation |
| states never triggered | neither — a property of the sequence |

## Recording sequence — template

    open the panel
      → traverse every element on each input device the game accepts
      → enter text in every field
      → cancel out one level at a time

A step whose precondition is absent is dropped and recorded `N/A` in the rule applicability table — never silently left out of the sequence.

## Extraction cadences

A coarse rate covers the sequence and its coverage; a native rate over a short window catches per-frame faults — the two do not substitute for each other, and four frames out of sixty discards exactly the shimmer the recording was made to catch. Commands per platform: `${CLAUDE_PLUGIN_ROOT}/docs/capturing-evidence.md`.
