# Good pineapple

## Challenge Overview

| Field | Value |
|---|---|
| Category | Misc |
| Challenge | [Kaspersky{CTF} challenge 2](https://ctf.kaspersky.com/challenges/2) |
| Artifact | `challenge.mkv` |
| Main idea | Recover an autostereogram depth plane and collect short-lived glyphs |
| Flag | `kaspersky{g3t_h1dd3n_b4d_4ppl3d_676767}` |

## Challenge Description

The video looked like monochrome television noise. The description warned that we might be looking at the wrong plane and that some details did not stay visible for long.

## Rejecting the Obvious Approaches

Simple frame differencing and bit-plane extraction produced noise rather than stable text. A more useful observation was that horizontal regions of each frame repeated with a small displacement. That is the structure of a random-dot autostereogram.

## Depth-Plane Recovery

1. Decode the MKV into frames without resizing or lossy recompression.
2. Divide each frame into small blocks or scan lines.
3. Compare every region with horizontally shifted copies over a limited disparity range.
4. Record the shift with the best correlation for each location.
5. Convert disparity into grayscale depth and smooth only enough to remove isolated errors.

The recovered depth frames revealed a hidden Bad Apple animation. This explained the “wrong plane” clue: the payload lived in disparity/depth, not in a color bit plane.

## Transient Character Layer

The animation was not the final answer. At regular moments, a single character appeared for only a few frames:

1. Compute temporal differences on the recovered depth video.
2. Flag short events that do not belong to the longer animation motion.
3. Save the relevant frame window for every event.
4. Stabilize or average those few frames so each glyph becomes readable.
5. Read the glyphs in chronological order.

This produced the text inside the final flag.

## Verification

The event sequence was ordered by source frame number, not by glyph similarity. Repeating the extraction with adjacent frames produced the same characters and eliminated one-frame compression artifacts.

## Flag

```text
kaspersky{g3t_h1dd3n_b4d_4ppl3d_676767}
```

