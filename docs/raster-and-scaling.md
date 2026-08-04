# Raster and scaling

The class of bug that passes as a "graphics glitch": pixel art that shimmers
in motion, a sprite that dims for no visible reason, a HUD that clips on the
one aspect ratio nobody tested. Each reads as a shader or driver problem
until traced back to a scaling, rounding or sort-order rule that was never
enforced.

## Preconditions

Establish which UI system the game actually uses before checking any rule
below — it is the first precondition and it governs every other row in this
file: a game whose UI draws no sprites at all, only vector shapes or a
DOM-like retained-mode tree, has none of the pixel-raster rows to satisfy.
This is read off the running game, not inferred from engine or genre.

Three further preconditions gate specific sections below:

- **Integer scaling, texel snapping and atlas padding** apply only when the
  game uses point-filtered (nearest-neighbour) sprites. Smooth-filtered art
  has no texel grid to snap to or bleed across — record `N/A — no
  point-filtered sprites` rather than applying a pixel-art rule to art that
  was never pixel art.
- **Ultrawide and 4:3 aspect-ratio checks** apply only on a platform that
  ships those ratios. A console locked to a single output ratio has nothing
  beyond it to check — record `N/A — platform ships 16:9 only`.
- **UI-scale extremes** apply only when the game exposes a UI scale option
  at all — many do not. A game with no scale slider has no "both ends" to
  test — record `N/A — no UI scale exposed`.

Each outcome goes into the rule applicability table as applicable or
`N/A — <reason>`; a "where one exists" hedge in prose does not satisfy that
requirement.

## Integer scaling and filtering

Point-filtered pixel art is scaled by an integer factor with
nearest-neighbour filtering, never by a fractional one. A 1.5× scale
resamples every source pixel onto a boundary that does not line up with the
destination grid, so a single-pixel outline blurs across two output pixels
and a checkerboard dither moirés instead of holding its pattern. Bilinear or
trilinear filtering does the same damage at any factor, including 1× — it
exists to smooth photographic textures, and applied to pixel art it blurs
the edges nearest-neighbour would have kept sharp.

## Texel snapping

Sprite positions round to texel boundaries; a position that lands between
two texels samples a blend of both, which reads as distortion on a still
and as shimmer once the sprite moves. Concrete instance, Core Keeper
1.2.1.4: sprites distort at positions exactly `k/16` for integer `k` —
that game renders its UI at 16 pixels per unit, so any position at a
multiple of `1/16` lands precisely on a texel edge. The value 16 is a fact
about that game's sprite-import setting, not a rule for pixel-art rendering
in general; a different point-filtered game snaps to whatever
pixels-per-unit it was authored at.

## Sorting

A render pipeline that sorts by depth (Z) rather than by an explicit layer
or order index makes two elements at equal Z collide, and the collision
surfaces as a colour or brightness fault rather than an ordering one —
which is why it gets misfiled as a shader problem. Concrete instance, Core
Keeper 1.2.1.4: a background element at the same absolute Z as the sprite
in front of it dims that sprite towards grey, with no change to either
sprite's colour data, and raising the front sprite's sorting order does
not lift it out — there, order decides mask clipping and Z decides depth,
so the fix is a distinct Z. Which key breaks the tie is a per-game fact.

## Reference resolution and UI scale

Layout is authored against a reference resolution and a scale factor,
never against absolute pixel sizes. A panel positioned and sized in raw
pixels holds together only at the resolution it was built on; a panel
positioned in reference units and multiplied by the scale factor keeps its
proportions at every resolution the scale factor is applied to.

## Aspect ratios and display scaling

Reference resolution alone does not prevent clipping or an unusable
composition — it guarantees proportions, not that every element still
fits. The layout is checked at the ratios and scales the target platform
actually ships: ultrawide 21:9 and 4:3 alongside 16:9, dynamic resolution,
OS-level display scaling, and — for the games that have one at all, per the
Preconditions record — both ends of the UI scale range. Display scaling
also blurs text and vector rasterization at a fractional DPI factor, on its
own terms and independent of any pixel-grid rule, so it is checked as its
own case rather than assumed covered once the pixel-art rules pass.

## Atlas padding

Adjacent sprites packed into one texture atlas bleed into each other under
filtering or mip-mapping: sampling near a cell's edge pulls in colour from
the neighbouring cell, a thin line of the wrong colour along edges that
looked correct in the source art. Padding — transparent or edge-extended
pixels between packed sprites — keeps that sample inside the sprite's own
cell. An atlas with zero padding, or a border thinner than the largest
filter or mip radius in use, still bleeds even though every sprite is
correct on its own.

## Engine specifics

How the rules above surface differs by engine, and by how a given game
built its UI on that engine. Read
`${CLAUDE_PLUGIN_ROOT}/docs/engines/unity.md` when the host or target
project runs on Unity — that appendix names the mechanisms (sprite
pixels-per-unit, camera sort mode, Canvas vs. SpriteRenderer) without
asserting their values, which keeps the rules above engine-free.

## Verification

Shimmer and distortion are motion faults: neither exists in a single
frame, so both are confirmed with a recording at native frame rate — move
a point-filtered sprite slowly across the screen and inspect the recording
frame by frame for edge shimmer, and pan the camera past a pair of sorted
elements to catch a dimming flicker as their order changes. Aspect ratios,
dynamic resolution, OS display scaling and UI-scale extremes are
configuration faults: confirm each with a lossless still per
configuration, taken after switching to that ratio, scale or DPI setting
and opening the busiest layout the UI has.
