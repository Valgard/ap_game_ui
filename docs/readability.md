# Readability

A UI can be well laid out and still fail the only test that matters: can the
player actually read it, at the moment they need to, under the conditions the
game itself creates — a moving background, a corner already claimed by three
other elements, a TV cropping the screen edge. Readability is checked against
those conditions, never against a static screenshot of the element alone.

## Preconditions

Establish which UI system the game actually uses before checking any rule
below — it decides what "readable" even means here: a bitmap font's fixed
glyph set scales differently than a vector one, and a hand-rolled HUD occludes
differently than a layout the platform docks and manages for you. This is
read off the running game, not inferred from the engine or the genre.

One further precondition gates a section below:

- **Safe areas, overscan and cutout** apply only when the game outputs to a
  TV, or ships on a device with a notch or cutout. A PC game bound to a
  monitor has no overscan margin to respect — record `N/A — no TV or cutout
  output` rather than reserving edge space nothing will ever crop.

## Comprehension budget

Readability is a budget sized to how fast the information must land, and that
budget is a design quantity, not a matter of taste.

- **A one-second budget and an untimed one are different problems.** Health,
  a poison stack, an incoming-hit warning: read in under a second or the hit
  lands unread. A crafting menu or a codex entry: read at leisure, so it can
  carry dense text and a low-priority spot on screen that a health readout
  cannot.
- **The budget drives size, contrast and placement — not taste.** A health
  number sized and placed like a crafting tooltip reads fine on a still of
  the menu it was actually designed for, and unreadable the instant it has to
  survive combat instead.

## Text over moving backgrounds

- **A carrier surface is required: an outline, a drop shadow or a translucent
  panel behind the text.** Contrast ratio alone is not enough, because the
  background is not one colour — text tuned for contrast against a dark cave
  wall turns unreadable the moment the camera pans over a bright sand tile a
  frame later, and no single ratio was ever wrong, the background just
  stopped matching it.

## Safe areas

- **TV overscan.** A HUD element placed flush against the screen edge renders
  in full in the editor's game view and is cropped by the actual TV's
  overscan margin, invisible on the only screen that matters.
- **Notch and cutout.** An icon anchored to the true top corner sits behind a
  phone's camera notch or a curved-corner cutout, unreadable until the safe
  margin is pulled inward.

## HUD occlusion

- **Corners are the most expensive real estate.** They are claimed first —
  minimap, currency, quest tracker — so a new element aimed at the same
  corner overlaps one of them instead of finding empty space.
- **The centre is off limits.** An element placed mid-screen sits on top of
  the crosshair, the combat and the player's own view of the action —
  occluding exactly what the player is looking at to play the game.

## Colour is never the sole carrier

- **Redundant encoding is required: shape, position or label alongside
  colour.** A red-versus-green-only distinction between "equipped" and
  "broken" is invisible to a colourblind player and unreadable in a
  colour-crushed TV picture mode; an icon shape, a fixed slot position or a
  text label carries the same distinction without the colour channel.

## Verification

- **Readability in motion** is confirmed with a recording at native frame
  rate: move the camera across the busiest background the game has while the
  text stays on screen, and watch whether the carrier surface holds up frame
  to frame — a still cannot show a fault that only appears as the backdrop
  changes under moving text.
- **HUD occlusion and safe areas** are confirmed with a lossless still per
  configuration: open every safe-area setting the platform ships — TV
  overscan on, notch present, cutout present — and inspect each corner and
  the screen edge for clipped or overlapped elements.
