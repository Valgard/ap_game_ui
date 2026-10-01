# game-ui

Two Claude Code skills for game UI and UX. They split along a single axis: **whose
form the interface carries.**

| Skill | Use when |
|---|---|
| `game-ui-design` | the form is yours to set — a new game, your own shipped game's interface, or a mod that deliberately replaces a host game's look |
| `game-ui-modding` | the form is someone else's — added panels, fields, menus or mod UI that must read as part of the host game |

They are two skills rather than one because their aesthetic directives are opposed.
A host-form skill has to suppress the designer's signature; an own-form skill has to
produce one. One skill carrying both would hedge on every decision it makes.

When a brief names a game and a UI surface without saying who owns the form — "HUD
for a space game", "inventory screen", "settings menu" — `game-ui-design` applies.
Defaulting to the host-form skill there would send it measuring a game nobody
specified.

Invocation names are `game-ui:game-ui-design` and `game-ui:game-ui-modding`.

## Installing

This repository is a plugin, not a marketplace, so it is installed through a catalog
entry. Add one to your own marketplace's `.claude-plugin/marketplace.json`:

```json
{
  "name": "game-ui",
  "description": "Game UI/UX design skills",
  "source": { "source": "url", "url": "https://github.com/Valgard/ap_game_ui.git", "ref": "1.0.0" }
}
```

Then declare the catalog and install from it:

```
claude plugin marketplace add <your-catalog-repo>
claude plugin install game-ui@<your-catalog-name>
```

Two things about that sequence are worth knowing. `marketplace add` writes only the
declaration — the runtime registry is rebuilt from it at the next session start, so
`install` answers `not found` until a session has started. And `claude plugin list`
shows only what is already installed; a freshly added catalog looks empty without
`--available`.

The `url` form is deliberate. A `github` source object validates, but across 325
entries in four installed marketplaces it had zero practical usage, so `url` is the
better-trodden path. The `ref` pins a release tag — drop it and the catalog follows
the default branch instead, so every push reaches installed copies.

## What each skill does

**`game-ui-design`** derives aesthetic direction, typography and layout from the
game's own fiction instead of from a UI trend. It names five looks it refuses to
arrive at by default — sci-fi HUD, fantasy parchment, mobile flat, indie retro
pixel, muted dark minimal — because those are what a model reaches for when it is
not thinking about the specific game. Choosing one *deliberately*, with the fiction
behind it, is fine; the test is whether the choice can be defended from the game
rather than from habit.

**`game-ui-modding`** treats every statement about the host game as a hypothesis
until it has been seen. "The panel background is `#1a2a2e`", "there is a bold
weight", "Escape closes this menu" — each sounds plausible, and a wrong guess here
is precisely what makes a mod read as foreign. The check is the running game: not
the editor, where camera, post-processing, UI scale and font fallback all differ;
not the wiki, which describes some version rather than the installed one; not
memory. A verification gate keeps what was measured separate from what was assumed,
and records both — including the cases where verification was not possible, and what
that costs.

## The shared constraint layer

Both skills reach the same reference files through `${CLAUDE_PLUGIN_ROOT}/docs/`.
These hold the constraints that are not aesthetic choices:

| File | Covers |
|---|---|
| `input-and-focus.md` | controller focus traversal — a controller is not a keyboard with different keys, and there is no cursor to fall back on |
| `readability.md` | legibility as a budget sized to how fast the information has to land |
| `raster-and-scaling.md` | pixel raster, texel alignment, the scaling bugs that pass as graphics glitches |
| `localization.md` | fixed-width boxes, RTL mirroring, CJK breaking, glyph coverage, plural rules |
| `verification-gate.md` | the lookup half of the gate — schemas and tables |
| `capturing-evidence.md` | OS-specific commands for stills and recordings |
| `engines/unity.md` | Unity ships several UI architectures; which one a project picked changes the advice |

Not every constraint applies to every game. A game shipping no right-to-left
language records that as `N/A` with the reason, rather than quietly dropping the
row — an unanswered check and an inapplicable one must not look alike.
