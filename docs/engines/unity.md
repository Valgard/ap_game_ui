# Unity appendix

Unity is not a UI architecture. It ships several, and a project picks one —
or builds its own. This file names the mechanisms whose values shape the
rules in the sibling docs, and says where to read each one. It never states
a value as a fact about Unity in general: every concrete number below is an
example from one named game and version, not a default.

## First: which UI system does this game use?

This is the question every other section here, and every rule in the
sibling docs, depends on. Unity's own UI framework is uGUI — `Canvas`,
`RectTransform`, the `UnityEngine.UI` namespace — but nothing forces a
project onto it: a game can draw its UI as `SpriteRenderer` objects on a
sorted layer, adopt a third-party retained-mode framework, or write its own
from scratch. How to tell: open the scene hierarchy in the Editor and look
for a `Canvas` versus a plain `GameObject` tree of sprites, or grep a
decompiled build for `UnityEngine.UI`, `SpriteRenderer`, and any vendor UI
namespace the game ships.

Two measured examples, same engine, disjoint answers. Core Keeper 1.2.1.4
draws its entire UI as `SpriteRenderer` objects on a dedicated GUI layer,
with a mod-side element base class standing in for `Canvas` — a survey of
ten UI mods for that game found `SpriteRenderer` plus a custom element base
in all ten, and zero use of `UnityEngine.UI`. Cities: Skylines 1 answers the
opposite way: its managed assembly carries 121 files under the
`ColossalFramework.UI` namespace (`UIComponent`, `UIPanel`, `UIView`) and
zero references to `UnityEngine.UI` or `SpriteRenderer`. Neither game uses
the framework Unity itself ships. Record the answer in the
rule-applicability table the sibling docs ask for — most rows in this file,
and in them, apply only once that answer is known.

## Sprite raster

MECHANISM: `spritePixelsToUnits` on the sprite importer sets how many
texture pixels make up one world unit — the texel size a
`SpriteRenderer`-based UI is drawn at. Filter Mode on the same texture's
importer chooses point (nearest-neighbour) sampling or bilinear/trilinear
smoothing. WHERE: the importer's Inspector panel, or the asset's `.meta`
file (`spritePixelsToUnits`, `filterMode`) when reading a project on disk
rather than through the Editor.

Example, Core Keeper 1.2.1.4: its `SpriteRenderer` UI is imported at 16
pixels per unit with point filtering and no mipmaps — texture-import
settings its own sheet's `.meta` template carries forward on every
regeneration. That value is a fact about this one game's import settings,
not a Unity constant; a different point-filtered game snaps to whatever
pixels-per-unit it was authored at, and a game whose answer to the first
section is not `SpriteRenderer` has no row here at all.

## Transparency sorting

MECHANISM: `TransparencySortMode` on the `Camera` component decides how
transparent renderers that tie on sorting layer and order break that tie —
by distance along a configurable axis, or not at all beyond the explicit
order. WHERE: the Camera Inspector's Rendering section (`Transparency Sort
Mode`, `Transparency Sort Axis`), or Project Settings > Graphics for the
project-wide default a camera can still override. Why it matters: two
renderers placed level along the sort axis collide, and the collision reads
as a colour or brightness fault — one sprite unexpectedly drawing over, and
dimming, another — rather than as an ordering fault, so it gets misfiled as
a shader problem.

Core Keeper 1.2.1.4's `SpriteRenderer` UI does not lean on that tie-break at
all: it stacks entirely through explicit per-renderer sorting orders — a
`SpriteMask` clip range of 40–55, a popup band of 56–63, individual
overrides down to a single footer element pinned below both — so an
element's place is a number authored in the prefab, not a position along an
axis. Cities: Skylines 1 sidesteps the mechanism a different way: since its
UI carries zero `SpriteRenderer` references, this camera setting governs
only the game world's transparent sprites and has no say over UI stacking
at all — that is purely `ColossalFramework.UI`'s own ordering model.

## Serialized data

MECHANISM: prefabs, scenes, and `ScriptableObject` assets are where authored
UI structure and values actually live — the layer a decompiled build's code
cannot show, because a prefab's field values are data, not instructions.
WHERE: the `.prefab` / `.unity` / `.asset` files themselves, readable as
YAML outside the Editor. Caveat: a build step can reserialize or re-template
these on export, so a value read from a source-tree asset is not guaranteed
to be the one a shipped build actually carries — compare against the built
or extracted asset when the two could differ.

Core Keeper 1.2.1.4 is a working instance of the mechanism, not just the
definition: every sprite's `spritePixelsToUnits`, Filter Mode, and 9-slice
border lives in that sprite's `.meta` file, and regenerating a sprite sheet
explicitly inherits its texture-import header from a template `.meta`
rather than from any code path — the asset file is the single source of
truth the tooling itself depends on.

## Verification

Establishing which UI system a game uses is neither a still nor a
recording — it is a code or scene inspection: open the hierarchy in the
Editor, or grep a decompiled build for `UnityEngine.UI`, `SpriteRenderer`,
and a candidate vendor namespace, and read the answer off what is present
or absent. Do this first; it decides which of the rows below even apply.

`spritePixelsToUnits` and Filter Mode are static importer settings —
confirm with a lossless still of the Inspector, or by opening the `.meta`
file directly; neither value changes over time, so no recording is needed
to catch a wrong one. `TransparencySortMode` is likewise a still of the
Camera Inspector's Rendering section, but whether a tie along its sort axis
actually makes two elements collide is a motion fault: catch it with a
recording, panning past the pair in question and watching for one dimming
or swapping order as the sort-axis distance changes. Serialized data is
checked with a still of the asset file's own content — compare the
source-tree `.prefab`/`.asset` against the built or extracted equivalent
when a reserialization step could have changed what shipped without
changing what is checked into source.
