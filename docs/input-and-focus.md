# Input and focus

A controller is not a keyboard with different keys. There is no cursor to
point with, so one abstract concept — focus — carries everything a mouse
pointer, a hover state and a click used to do separately. Focus must always
exist somewhere, must move in well-defined ways, and must not silently
disappear when the thing under it does.

## Preconditions

Before any rule below is checked, establish which UI system the game actually
uses. It decides what "focus" even means here: a custom retained-mode UI, a
standard toolkit's widget tree, and a hand-rolled immediate-mode overlay each
route input and highlight selection differently, so a rule that holds for one
may not exist at all in another. This is read off the running UI, not
inferred from the engine or the genre.

Two further preconditions gate the rest of this file:

- **Controller focus traversal and device-specific button prompts** apply only
  if the game accepts a gamepad. A game with no gamepad support has no "Focus
  position" rules to satisfy — record `N/A — no gamepad support` rather than
  half-applying them to a mouse-only menu.
- **Text entry vs. gameplay input, and the on-screen keyboard path,** apply
  only if some UI surface takes typed input — a chat box, a rename field, a
  search filter. A UI with no text field has nothing in "Text entry" to check;
  record `N/A — no typed input in this UI`.

A rule whose precondition is absent does not exist for this project. Recording
it `N/A` with a reason is what separates "not applicable" from "not checked."

## Focus position

- **Neighbours are explicit, not derived.** Geometric nearest-neighbour
  computation fails on irregular grids: a two-column "equip" button sends
  "nearest by distance" into the wrong slot above it, or a disabled gap in a
  5-wide inventory row leaves the cursor bouncing between two cells.
- **Initial focus is a decision.** A menu opening with nothing focused means
  the first gamepad press does nothing, or a confirm press fires on whatever
  was focused underneath before the menu opened.
- **Hover and focus are two states, not one.** Reusing hover-highlight code as
  the focus indicator leaves nothing focused once the mouse leaves the panel,
  and a gamepad-selected row loses its highlight whenever the mouse rests
  elsewhere.
- **Scroll-follow is pivot-correct.** When the newly focused row sits outside
  the viewport, the scroll must move to include it — always snapping to the
  top edge still clips a tall row focused from below, so the correction must
  account for which edge is nearer the viewport boundary.
- **Prompts follow the active device.** A player who switches from gamepad to
  keyboard and still sees an Xbox glyph on the confirm prompt has no way to
  know which key does it.
- **Nothing is reachable by hover only.** A tooltip or flyout that only opens
  on mouse hover, with no focus-driven equivalent, is invisible to gamepad and
  keyboard-only play.

## Focus lifecycle

Focus has a lifecycle, not only a position. A filter field above a scrolling
list exercises every rule here at once, because filtering *is* removing the
focused row — the normal path through this section, not an edge case.

- **Restoration after a nested panel closes.** Cancelling a confirmation
  dialog should return focus to the row that opened it; returning it to
  nothing, or to the top of the list, forces the player to re-navigate from
  scratch after every cancelled action.
- **The focused row can disappear.** Filtering it out, removing it, disabling
  it or reordering the list are the same problem: focus needs a stated rule
  for where it goes next — the next visible row at the same index, or the
  first item if none remain — instead of vanishing or landing at random.
- **Disabled and hidden controls are skipped consistently in every
  direction.** Skipping a disabled row correctly moving down but landing on
  it moving up means the skip logic was written for one direction only.
- **A device switch mid-focus keeps the selection.** Tapping a keyboard key
  while "Craft" is gamepad-focused should swap the prompt glyph in place, not
  reset focus to the first slot as if the panel had just opened.

## Text entry and input capture

- **Gameplay input is suppressed while a field holds focus.** Otherwise
  typing a rename or chat message also walks the character, because movement
  reads raw key state instead of asking whether a text field owns input.
- **Cancel priority is defined per nesting level.** A rename field inside an
  inventory panel needs a stated answer for what one Escape press does first —
  clear the field, close the panel, or open the pause menu — instead of the
  outermost handler winning by accident and discarding a half-typed value.
- **IME and composition must work.** A field that fires an input event per
  raw keystroke instead of waiting for IME composition to commit never
  produces a correct CJK character — the field silently accepts only Latin
  input, a complete failure for those languages, not a partial one.
- **Controller-only and handheld sessions need an on-screen keyboard path.** A
  rename field reachable with a gamepad but no on-screen keyboard invocation
  cannot be filled in without external hardware.
- **Every new binding is checked against the host's existing bindings before
  it is claimed.** An "interact" key repurposed for "favorite item" produces
  a collision that fires both actions, or silently shadows the original one.

## Verification

- Focus traversal (neighbours, initial focus, hover-vs-focus, pivot-correct
  scroll-follow) is confirmed by traversing the full grid or list with only
  the target device, recorded start to finish — a recording, because a wrong
  neighbour or a clipped row shows up as a jump between frames, never inside
  one.
- A device switch is confirmed by switching input mid-recording: the prompt
  glyph swaps and the focused element does not move, in one continuous take.
- The lifecycle rules are confirmed by opening and closing a nested panel, and
  by typing into a filter field until the focused row disappears, while
  recording throughout — restoration and the disappearing-row rule are both
  "did focus move between two moments," not a property either moment holds.
- Input capture is confirmed by typing into every text field while recording,
  including one pass through a CJK input method, and by exercising the
  on-screen keyboard path on a controller-only session — a recording in every
  case, since a keystroke sequence has no meaningful single frame.
- The binding-collision check is the one static fact here: compare the new
  binding against the host's control list side by side in a lossless still.
