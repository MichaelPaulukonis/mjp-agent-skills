---
name: image-frames-to-video
description: Use when creating video from numbered image frames in a folder, using tool.sh or direct ffmpeg. Triggers on - "make video from frames", "frames to video", "image sequence to mp4", "tool.sh --video".
license: MIT
compatibility: Requires ffmpeg (libx264). The tool.sh path also needs tool.sh from https://github.com/MichaelPaulukonis/image-processing-shell plus ImageMagick.
metadata:
  author: MichaelPaulukonis
  homepage: https://github.com/MichaelPaulukonis/mjp-agent-skills
---

# Image Frames to Video

## Overview

Two paths: `tool.sh` wrapper (uses `ROOT`-relative paths) or direct `ffmpeg` (absolute paths, more flexible). Direct ffmpeg is simpler for one-off or non-standard `ROOT` locations.

`tool.sh` comes from [image-processing-shell](https://github.com/MichaelPaulukonis/image-processing-shell). The direct-ffmpeg path needs nothing else.

## Quick Reference

| Want | Use |
| ------ | ----- |
| Quick video, frames in `~/projects/images/` | `tool.sh` |
| Absolute path or custom location | Direct `ffmpeg` |
| Multiple FPS variants at once | `tool.sh` (omit `--fps`) |
| Single FPS, full control | Direct `ffmpeg` |

## Direct ffmpeg (Recommended)

```bash
# Basic: glob all PNGs, 10fps, output to same folder
ffmpeg -r 10 -f image2 -pattern_type glob \
  -i "/path/to/frames/*.png" \
  -vcodec libx264 -crf 17 -pix_fmt yuv420p \
  "/path/to/frames/output.10.720.mp4"

# With square padding/scaling (matches tool.sh behavior)
ffmpeg -r 10 -f image2 -pattern_type glob \
  -i "/path/to/frames/*.png" \
  -vf "scale=720:720:force_original_aspect_ratio=decrease,pad=720:720:(ow-iw)/2:(oh-ih)/2" \
  -vcodec libx264 -crf 17 -pix_fmt yuv420p \
  "/path/to/frames/output.10.720.mp4"
```

**Example - frames in a genart-output folder:**

```bash
ffmpeg -r 10 -f image2 -pattern_type glob \
  -i "$HOME/projects/images/genart-output/*.png" \
  -vf "scale=720:720:force_original_aspect_ratio=decrease,pad=720:720:(ow-iw)/2:(oh-ih)/2" \
  -vcodec libx264 -crf 17 -pix_fmt yuv420p \
  "$HOME/projects/images/genart-output/dragline.burst.10.720.mp4"
```

## tool.sh Wrapper

`ROOT` is the directory holding your frame folders, not where `tool.sh` lives. It is hardcoded to `~/projects/images` at the top of `tool.sh` - edit that line to change it. All `--target` / `--destination` paths are relative to `ROOT`.

```bash
cd /path/to/image-processing-shell

# Single FPS
./tool.sh \
  --target genart-output \
  --destination genart-output \
  --name dragline.burst-20260620193745 \
  --video --fps 10 --resolution 720

# All FPS variants (5, 10, 20, 30) - omit --fps
./tool.sh \
  --target genart-output \
  --destination genart-output \
  --name dragline.burst-20260620193745 \
  --video --resolution 720
```

Output: `$ROOT/$DESTINATION/$NAME.$FPS.$RESOLUTION.mp4`

**tool.sh limitations:**

- Hardcoded to `*.png` (not `--suffix` configurable for video)
- Requires frames in `$ROOT`-relative path
- `--verbose` prints the ffmpeg command before running

## Frame Naming Patterns

A typical pattern: `{name}.frame-NNNN.png` (4-digit zero-padded)

ffmpeg glob `*.png` handles any naming as long as filenames sort correctly alphabetically.

## Common Mistakes

- `tool.sh`: frames outside `~/projects/images` → `ROOT` is hardcoded; use direct ffmpeg or edit `ROOT` in `tool.sh`
- Resolution not even → libx264 requires even dimensions; the padding filter handles this
