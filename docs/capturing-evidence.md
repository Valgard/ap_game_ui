# Capturing evidence

Commands are OS-specific and therefore live here rather than in a skill
body. The cadence rule — coarse rate for sequence and coverage, native rate
for per-frame faults — is in
`${CLAUDE_PLUGIN_ROOT}/docs/verification-gate.md`; this file is only how the
artifacts get produced.

## macOS

- **Lossless still:** `screencapture -x <out>.png` — PNG, no shadow, no
  shutter sound.
- **Recording:** `screencapture -v -V<seconds> -k <out>.mov` — `-k` draws
  clicks and key presses on screen, `-V` bounds the length so a forgotten
  recording does not run unbounded.
- **Sequence frames** (coarse rate, for the recording-sequence template):
  `ffmpeg -i <out>.mov -vf fps=4 seq-%03d.png`.
- **Per-frame window** (native rate, for shimmer and other per-frame
  faults): `ffmpeg -i <out>.mov -ss <t> -t 1 shimmer-%03d.png` — one second
  at the source frame rate, centred on the moment in question.

## Other platforms

An entry for another platform must supply all four of the above: a
lossless-still command, a recording command with an explicit length bound,
and both extraction cadences at the rates the cadence rule requires. Add an
entry when a project actually needs one — do not fill this section with an
untested guess at what the equivalent command might be.

## Reading recordings

A model reviews extracted frames, not the source video — a recording file
is not something a text-based review step can inspect directly. Name frames
so the axis is legible from the filename alone: `seq-*` for the coarse
sequence pass, `shimmer-*` for the native-rate window, so a reviewer never
has to open a file to know which cadence produced it.
