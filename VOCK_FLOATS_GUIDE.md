# VockFeatures config guide

What each setting does, in plain terms. For the technical design record (why things work
this way internally, code references, commit history) see `VOCK_FLOATS.md` instead — this
file is just "what does turning this knob do."

## What this is

"Floats" are the text lines that pop up over an NPC's head — combat barks, ambient chatter,
flavor lines from NPCs without a full dialogue window. Normally they're silent text only.
This adds: real voice-over audio for floats that have it, volume that fades with distance,
walls/scenery muffling a float, a censor bleep for filtered lines, optional text garbling
for lines you can barely hear, and voiced narration for Pip-Boy holodisks on their own
dedicated audio channel.

## Turning it on

**`fission.cfg`**, under `[enhancements]`:

```ini
[enhancements]
StrictVanilla=0
VockFeatures=0
```

`VockFeatures` is **off by default** — `VockFeatures=1` turns the whole feature set on
(floats and Pip-Boy narration alike), `VockFeatures=0` turns it all off. If `StrictVanilla=1`
is set, that overrides `VockFeatures` off no matter what it's set to.

**`data/game.cfg`**, under `[vock-features]` — the individual settings:

```ini
[vock-features]
FloatAudioChannels=8
FloatDistancePerPerception=2
FloatObstructionDampening=50
FloatEvictionPolicy=0
FloatAudio=1
FloatCensorBleep=1
FloatVolume=32767
TextScramble=0
TextScrambleDistancePerPerception=2
TextScrambleObstructionDampening=50
TextScrambleChars=#%&*~^
PipboyAudio=1
PipboyVolume=32767
```

Every key here starts with the feature it belongs to: `Float*` for NPC float audio,
`TextScramble*` for the on-screen text garbling, `Pipboy*` for holodisk narration. They're
three independent features that happen to share one config section — none of the `Float*`
settings affect `TextScramble`, and vice versa, even though both are about the same floating
text lines.

## Each setting

## Float audio

These control the actual voice-over audio that plays when an NPC's float has a voice file.

### FloatAudioChannels
**Default: `8`**

How many floats can have voice audio playing at the same time. Each NPC only ever takes up
one channel no matter how many lines it fires — a new line from an NPC that's already
speaking just replaces its own old line. Raise this if floats are getting cut off in busy
scenes with lots of talking NPCs at once.

### FloatDistancePerPerception
**Default: `2`**

Controls how far a float's voice carries before it's silent. The actual range is
**your Perception stat × this number**, in tiles. With the default of `2`: a float fades out
completely by 2× your Perception in tiles. The fade is a straight linear ramp — full volume
right next to the speaker, quieter as you move away, silent at the max range.

Raise this to hear floats from farther away. Lower it to make the game quieter/closer-range.
Independent of `TextScrambleDistancePerPerception` below — the text garbling has its own
range, not derived from this one.

### FloatObstructionDampening
**Default: `50` — range `0`–`100`**

How much a solid wall or piece of scenery between you and the speaker muffles a float's
*audio*, as a percentage. `0` = walls don't matter, a float sounds the same whether it's
blocked or not. `100` = a blocked float is completely silent. Anything in between scales it
down proportionally. Only walls/scenery block sound this way — other NPCs standing between
you and the speaker don't count.

Independent of `TextScrambleObstructionDampening` below — a wall can muffle what you hear
without necessarily garbling what you read, or vice versa, if you tune the two differently.

### FloatEvictionPolicy
**Default: `0` (Vanilla/no eviction)**

What happens if every channel (see `FloatAudioChannels`) is already busy and a new float
wants to play:

- **`0` — Vanilla**: the new float just doesn't play. Whatever's already playing keeps going
  untouched.
- **`1` — Oldest**: the float that's been playing longest gets cut off to make room for the
  new one.
- **`2` — Furthest**: whichever currently-playing float's speaker is farthest from you gets
  cut off — but only if the new float's speaker is actually closer. This never makes things
  quieter overall; it just swaps a distant voice for a closer one.

### FloatAudio
**Default: `1` (on)**

Master toggle for whether floats that have a voice-over file actually play it. Turn this off
to keep the distance/text behavior but go back to silent floats.

### FloatCensorBleep
**Default: `1` (on)**

A line that got caught by the profanity filter never plays its real audio, no matter what
this is set to. This only decides what happens *instead*: `1` = you hear a short censor
"bleep" tone. `0` = you hear nothing at all for that line.

### FloatVolume
**Default: `32767`** (max, i.e. 100% — no reduction on top of your SFX slider)

A volume multiplier applied on top of your normal Sound Effects volume slider. This can't
make floats louder than your SFX volume allows, and if you mute SFX entirely, floats go
silent too — it's a multiplier on that slider, not a separate volume channel.

## Text scrambling

A separate feature from float audio above — it garbles a float's on-screen *text*,
independently of whether that float has a voice file at all or plays it.

### TextScramble
**Default: `0` (off)**

Garbles the floating text on screen based on its own distance/obstruction math (see
`TextScrambleDistancePerPerception`/`TextScrambleObstructionDampening` below) — not the
audio settings above. Close and clear = text reads fine. Far away or blocked = text degrades
into noise characters — a float you can barely hear also gets hard to read, instead of being
perfectly legible from anywhere on screen.

Text reads perfectly clean out to 3/4 of its own range, then progressively garbles more over
the last quarter, until it's fully scrambled at the range itself. Only letters get replaced —
spaces and punctuation are left alone, so you can still tell where words start and end even
when heavily garbled.

### TextScrambleDistancePerPerception
**Default: `2`**

Text's own version of `FloatDistancePerPerception` above — same **Perception × this number**
range, same straight linear fade, but tuned independently of the audio range. Same default
(`2`) as the audio setting, but the two aren't linked: raising one doesn't affect the other.

### TextScrambleObstructionDampening
**Default: `50` — range `0`–`100`**

Text's own version of `FloatObstructionDampening` above — same meaning, same wall/scenery
raycast, but its own independent value. Same default (`50`) as the audio setting, but again,
not linked.

### TextScrambleChars
**Default: `#%&*~^`**

The pool of characters `TextScramble` picks from to replace letters. Change this to whatever
you want the garble to look like, e.g. `TextScrambleChars=*$%^`. If you leave this blank, it
falls back to the default set above.

## Pip-Boy holodisk narration

### PipboyAudio
**Default: `1` (on)**

Master toggle for voiced Pip-Boy holodisk narration specifically. Independent of
`FloatAudio` above — you can have voiced NPC floats without voiced holodisks, or vice
versa. Holodisk narration plays on its own dedicated audio channel, separate from both NPC
floats and dialogue speech, so it can't be interrupted by (or interrupt) either one. Audio
files for this feature live under `sound/pipboy/`, a sibling of `sound/speech/`
rather than a subfolder of it, since holodisk narration has no critter/head
behind it and isn't dialogue.

### PipboyVolume
**Default: `32767`** (max, i.e. 100% — no reduction on top of your Speech slider)

A volume multiplier applied on top of your normal Speech volume slider, just for Pip-Boy
holodisk narration. Same relationship `FloatVolume` above has to the SFX slider, but layered
onto Speech instead since holodisk narration is spoken dialogue, not an ambient effect.
