---
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
---

# Game UI design

Approach this as the design lead of a small studio, hired for a point of
view: the client wants an interface that could belong to no other game.
New title, your own shipped game's next revision, or a mod-overhaul — the
goal is the same: an identity of its own, not a competent default.

## If the brief is a mod that must blend in

If the UI must instead blend into someone else's game — read as something
that game's own team could have shipped — this is the wrong skill; use
`game-ui-modding` instead. Everything whose form is yours stays here: a new
game, your own shipped game's interface, an overhaul replacing a host's look
with one of its own — see *When the target is an existing game* below.

## Ground it in the game's fiction

Before any palette or layout decision, name the game's world: its
materials, its instruments, its vernacular. A blacksmithing sim's crafting
menu and a starship's maintenance terminal differ because the fiction
supplies different objects, not by taste. This is stronger leverage than
on the web: a game's fiction licenses treatments a marketing site has no
premise for — a burnt parchment edge, a telemetry-noisy status readout.
Build every token from the subject in hand, not from "UI in general".

## Pick the diegetic layer deliberately

Decide, and state, which layer the interface lives in: **diegetic**
(inside the fiction, a character could see it — a wrist terminal); **meta**
(styled by the fiction but not physically present — a blood-spatter damage
vignette); **spatial** (in the 3D world but outside the fiction — a
floating waypoint marker); or **non-diegetic** (the traditional HUD
overlay). Modding inherits this choice from the host; here it is made
once, up front, governing every later call about what the interface shows.

## Not these five looks

Five recurring shapes to recognize — the game-UI counterpart of the web's
own AI-generated defaults:

1. **Sci-fi HUD** — semi-transparent dark panel, cyan glow outline, thin
   technical all-caps sans, corner brackets as decoration.
2. **Fantasy parchment** — beige texture, serif display, gold ornament
   frame, wax-seal buttons.
3. **Mobile flat** — white icons on 60% black, thick rounded buttons,
   saturated green/red for yes/no, everything centered.
4. **Indie retro pixel** — midnight-blue panel, 1px white outline, 8×8
   bitmap font, hearts as a health meter.
5. **Muted dark minimal** — desaturated dark cards, hairline dividers,
   restrained neutral sans, sparse monoline icons, thin progress bars.

This is an open list, not a taxonomy, and **no frequency claim is made**:
there is no corpus behind it, only shapes to recognize. What marks a look
as a default rather than a choice is that nothing in it points at this
subject — the same treatment would serve a farming sim and a survival
horror game equally well. Cluster 5 deserves the most suspicion: restraint
reads as deliberation, so it is the one most likely to survive a
self-review the other four would fail. When a design passes self-review
and still feels templated, name the shape and add it here.

## Process

1. **Invent** a token system: palette as 4–6 named hex values, 2+ type
   roles, a layout concept, a signature element — plus the diegetic layer
   and the input model (which devices, and what focus looks like on each).
2. **Self-review** the plan against the five clusters above; anything that
   would serve a different subject equally well gets revised, with what
   changed and why written down.
3. **Build** to the revised plan, deriving every colour and type decision
   from the token table rather than improvising later.
4. **Critique**: does it read as unmistakable — could this interface
   belong to any other game?

## Constraints that are not yours to choose

The mechanics layer below is not a style question and is shared with
`game-ui-modding`; it decides right from wrong the same way in both
skills. Read each file for the reason given, before the work it governs:

- Before designing focus order, button prompts or any text-entry field,
  read `${CLAUDE_PLUGIN_ROOT}/docs/input-and-focus.md`.
- Before sizing HUD text or placing anything near a screen edge or corner,
  read `${CLAUDE_PLUGIN_ROOT}/docs/readability.md`.
- If any art is pixel-based or point-filtered, read
  `${CLAUDE_PLUGIN_ROOT}/docs/raster-and-scaling.md` before finalizing
  sprite positions, scale or layer order.
- Before locking a box size, a font, or any string shipping in more than
  one language, read `${CLAUDE_PLUGIN_ROOT}/docs/localization.md`.
- Consult `${CLAUDE_PLUGIN_ROOT}/docs/engines/unity.md` when the target
  runs on Unity, to establish which UI system the project actually uses
  before any of the above is applied to it.
- Read `${CLAUDE_PLUGIN_ROOT}/docs/verification-gate.md` before writing the
  token table — it holds the provenance and applicability schemas, and the
  capture commands one hop further in `capturing-evidence.md`, reached from
  there once its cadence rule has been read; never pointed at directly here.

Preconditions — gamepad support, typed input, a UI scale — are established by
looking, same as any value, except greenfield decides them rather than
discovering them. Before implementation starts, record each decision in the
applicability table at `${CLAUDE_PLUGIN_ROOT}/docs/verification-gate.md`: an
unwritten decision cannot be checked against later.

## When the target is an existing game

The existing game is either someone else's, in the overhaul case, or your
own already shipped — a HUD clipping on ultrawide, gamepad focus broken in
the options menu, a language being added. Either way the aesthetics stay
free and the existing build's mechanics stay binding: font availability,
the input system, the resolution model, and the fiction's existing frame.

Provenance attaches to where a value came from, not to which skill produced
it: invented values (palette, type, layout, signature) carry none, being
decisions with no source to cite, while a mechanics fact about a build that
already exists is a finding and carries full provenance. Four criteria decide
when that record is finished:

1. **Completeness** — every such fact the implementation relies on has a
   row; a value in the code with no row is a defect, not an omission.
2. **Identifiability** — the row names artifact, game version and the
   settings that affect the value, and the artifact exists where it says.
3. **A reason, not a shrug** — `ASSUMED` states why verification was
   impossible and what would settle it, in the answer and not only in the
   table. "Not checked" without a reason is incomplete, not `ASSUMED`.
4. **Saturation limit** — if the load-bearing facts are *all* `ASSUMED` the
   gate is **not passed**: the result is a draft and is reported as one. An
   unmarked value is never legal at all — a guess formatted like a
   measurement is the failure the gate exists to prevent.

New grammar for a single subsystem — the inventory alone, the map alone —
belongs here regardless of scope; it is never modding at reduced size.
What the partial case adds is the seam: where new grammar meets the
untouched rest is the quality marker, the one place a player sees both.

## Restraint and self-critique

Spend the boldness in one place. Once the signature element is set,
everything around it stays quiet — cut any decoration not doing work for
this brief. Chanel's rule applies here too: before shipping, look again
and remove one accessory. Build to a quality floor without announcing it:
every input device covered, colour never the sole carrier of information,
motion that respects a reduced-motion setting where offered.

## Writing

Write in the interface's own voice, not a narrator's. Name controls by
what the player does with them, not the system underneath — a player
manages a crew, not an `NPCManager`. Default to active voice: a button
says what happens when pressed, and the result keeps that word — "Craft"
produces a log line reading "Crafted," never "Item created." An error
states what happened and how to fix it, without apologizing. An empty
inventory or an unexplored map is an invitation to act, not a blank state
to tolerate.
