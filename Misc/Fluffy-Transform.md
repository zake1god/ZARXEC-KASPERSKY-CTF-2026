# Fluffy Transform

## Challenge Overview

| Field | Value |
|---|---|
| Category | Misc |
| Challenge | [Kaspersky{CTF} challenge 3](https://ctf.kaspersky.com/challenges/3) |
| Artifact | `challenge.wav` |
| Main idea | Demodulate three amplitude envelopes and decode their motion timing as Morse |
| Flag | `kaspersky{P4WS_1N_TH3_SP3CTRUM}` |

## Challenge Description

The description referred to three voices sharing one throat: one smooth, one with sharp corners, and one that kept changing direction. It also hinted that both movement and timing were significant.

## Spectral Analysis

A spectrogram showed three stable carriers centered near:

```text
900 Hz
2.2 kHz
4.5 kHz
```

Their frequencies and phases were essentially fixed. The changing information was in amplitude, so treating the challenge as frequency-shift keying or phase modulation did not explain the data.

## Envelope Recovery

1. Band-pass the mono recording around each carrier.
2. Demodulate or take the analytic-signal magnitude to obtain three smooth amplitude envelopes.
3. Low-pass the envelopes to remove carrier ripple.
4. Normalize all three channels to a comparable range.
5. Interpret the three amplitudes as `(x, y, z)` coordinates over time.

Plotting that path produced a cat-like figure, matching the “fluffy” clue. The image confirmed the channel interpretation, but it was not itself the flag.

## Timing Layer

The useful signal was whether the 3D point was moving or stationary:

1. Differentiate the three coordinates and combine them into a motion magnitude.
2. Threshold that magnitude into moving and still states.
3. Ignore the first second of setup; at exactly one second, the sequence aligns to 75 ms units.
4. Convert moving runs of one and three units into Morse dots and dashes.
5. Interpret still runs of one, three, and seven units as intra-character, character, and word gaps.

The timing decoded to:

```text
P4WS_1N_TH3_SP3CTRUM
```

## Verification

The run lengths stayed very close to integer multiples of 75 ms across the whole message. That global consistency was stronger evidence than attempting to read the 3D drawing by eye.

## Flag

```text
kaspersky{P4WS_1N_TH3_SP3CTRUM}
```

