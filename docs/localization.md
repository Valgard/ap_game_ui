# Localization

A UI built and tested in one language carries assumptions a second language
breaks for free: a box sized to the source string, a sentence assembled by
concatenation, a font whose glyph set stops at the source alphabet. None of
these are translation problems — they are UI defects a translation exposes.

## Preconditions

Establish which UI system the game actually uses before checking any rule
below — a fixed-atlas bitmap font and a TextMeshPro-style rasterizer fail
differently on the same gap in the character set. This is read off the
running game, not inferred from the engine or the genre.

Two further preconditions gate the Beyond Latin scripts section below:

- **Right-to-left mirroring and contextual shaping** apply only when the game
  ships an RTL language (Arabic, Hebrew, …) — record `N/A — no RTL language
  shipped` otherwise, rather than mirroring a layout no string will ever need.
- **CJK line breaking and IME composition** apply only when the game ships a
  CJK language (Chinese, Japanese, Korean) — record `N/A — no CJK language
  shipped` otherwise.

Each outcome goes into the rule applicability table as applicable or
`N/A — <reason>`.

## Expansion

German runs roughly 30% longer than English for a short UI string — a button
label, a stat name. That is a useful order of magnitude for how tight a
fixed-width box can be, not a ranking of the worst-case language: Finnish and
Russian regularly run longer than German on the same string, and CJK fails on
entirely different grounds (density and glyph coverage, not length). Which
language produces the longest rendering is a property of the individual
string, not of a language in general, so a box sized against one presumed
worst-case language still clips a different string in a different language.
The check covers every shipped language, not the language that was longest
last time.

## Glyph coverage

A bitmap font or a baked atlas font carries a fixed character set decided at
build time, not at translation time. A glyph outside that set does not fail
the build — it silently substitutes a fallback face for the missing character
only, so a string that was legible up to that point changes face mid-string.
Concrete instance, Core Keeper 1.2.1.4: one of the game's font variants
carries digits but no umlauts, so a string mixing a number with an umlauted
letter renders the two in visibly different faces, the switch landing
wherever the umlaut falls. The font has to be checked against the full
character inventory of every shipped language, not against whatever
characters happened to appear while testing in English.

## String assembly

Building a string by concatenating fragments — `"You found " + itemName +
"."` — bakes in an English word order other languages do not share; a
translator can only fix the sentence by rewriting code, because the
fragments carry no grammar for the translation file to change.

Placeholders inside a format string must be reorderable independent of their
source position: `"{0} equipped by {1}"` has to let a translation place them
as `"{1} hat {0} ausgerüstet"` without touching code. A pipeline that
hardcodes `{0}` before `{1}` cannot produce a grammatical translation for a
language with a different subject-object-verb order.

Plural rules are language-dependent, not binary: English has two plural
forms, Russian has three, Arabic has six. A hardcoded `count == 1 ? "item" :
"items"` check has no branch for anything between English and Arabic, so
those translations get a form that is wrong for some count.

## Beyond Latin scripts

Right-to-left languages mirror the layout, not merely the text direction:
focus order, icon direction and progress-bar fill direction all invert along
with the text. A health bar that keeps draining left-to-right under
right-to-left text, or a focus order that still tabs left-to-right through a
mirrored row of buttons, reads as broken rather than translated.

Arabic requires contextual shaping: each letter takes one of up to four forms
depending on its neighbours, and a letter rendered in isolation is often a
different, unreadable glyph, not a stylistic miss. What breaks Arabic is
character-by-character rendering without a shaping step first — not the use
of a texture atlas as such. An atlas stays compatible as long as it stores
the already-shaped glyph forms and selection happens after shaping runs; the
fault is skipping the shaping pass, not the storage mechanism downstream of
it — "the atlas doesn't support Arabic" sends the fix at the wrong component.

CJK breaks lines per character rather than per word: Chinese and Japanese
carry no spaces to break on, so a word-wrap rule built for space-delimited
languages wraps mid-character-cluster or never wraps at all. Mixed-direction
strings — an RTL sentence embedding an LTR item name, or a CJK sentence
carrying an English brand name — need their own test case: passing the RTL
and CJK checks in isolation does not establish that both behave correctly
nested inside one string.

## Raw term keys

A UI element showing its raw term key — `ui.settings.motion_blur` instead of
"Motion Blur" — is a registration fault, not a design or translation
problem: the term was never added to the localization table for this build,
or the key was mistyped between authoring and lookup. It needs the term
registered, not a translator; filing it as a translation gap leaves the
broken lookup in place while the raw key keeps showing in every language,
including the source one.

## Verification

Expansion, glyph coverage, shaping and mirroring are all static states —
none changes over time the way focus traversal or texel shimmer does — so
each is confirmed with a lossless still, one per shipped language, never a
recording. Switch the UI language, open the screen with the most and longest
strings for it (typically settings or the crafting list), and inspect every
string for clipping, a fallback-face seam, an unmirrored icon or an
unshaped Arabic run.

The longest known string is an additional wrap test on top of that, never a
substitute for it: one string surviving at its widest says nothing about the
others shipping in that language, so per-language coverage stays the
baseline and the longest-string pass is extra.
