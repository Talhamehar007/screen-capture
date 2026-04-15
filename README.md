# snapstream

`snapstream` is a standalone, minimal, **screen-capture-only** CLI that writes **ffmpeg-encoded video chunks**.

It keeps only the capture pipeline and intentionally excludes higher-level platform features.

## What It Does

1. Enumerates displays.
2. Captures frames from a selected monitor at `--fps`.
3. Streams raw RGBA frames into `ffmpeg`.
4. Encodes compact MP4 chunks (`libx265`/`libx264`) for storage-efficient recording.

## What It Does Not Do

- No OCR
- No audio capture/transcription
- No AI or summarization
- No HTTP server / WebSocket APIs
- No cloud sync

## Why Video Chunks (Not Images)

This tool is optimized for storage efficiency and long-running capture.

- Images (`.jpg/.png`) are simple but quickly consume disk.
- Encoded MP4 chunks significantly reduce footprint.
- Chunked output is resilient for long sessions and easier to archive.

## Requirements

- Rust `1.92.0` (see `rust-toolchain.toml`)
- `ffmpeg` available on PATH (or pass `--ffmpeg-path`)

Install ffmpeg examples:

- macOS: `brew install ffmpeg`
- Ubuntu/Debian: `sudo apt install ffmpeg`
- Windows: install ffmpeg and ensure `ffmpeg.exe` is on PATH

## Build

From `snapstream/`:

```bash
cargo build --release
```

Binary:

```text
target/release/snapstream
```

## Quick Start

List monitors:

```bash
cargo run -- --list-monitors
```

Capture with defaults:

```bash
cargo run -- --fps 2 --directory ./captures
```

Stop with `Ctrl+C`.

## CLI Reference

```text
snapstream [OPTIONS]
```

| Flag | Type | Default | Description |
|---|---|---|---|
| `--fps` | float | `2.0` | Capture rate (capped at 30 FPS) |
| `--directory` | path | `screenpipe-captures` | Output directory for video chunks |
| `--output-dir` | path | alias | Compatibility alias for `--directory` |
| `--monitor-id` | int | auto | Target monitor ID |
| `--chunk-seconds` | int | `30` | Chunk rotation interval |
| `--video-quality` | enum | `balanced` | `low`, `balanced`, `high`, `max` |
| `--codec` | enum | `h265` | `h265`, `h264` |
| `--frames` | int | unlimited | Capture exactly N frames then exit |
| `--ffmpeg-path` | path | `ffmpeg` | ffmpeg binary path |
| `--list-monitors` | bool | `false` | Print monitors and exit |

### Quality Mapping

`--video-quality` maps to encoder parameters by codec:

- `--codec h265`:
  - `low` → CRF 32, preset ultrafast
  - `balanced` → CRF 23, preset ultrafast
  - `high` → CRF 18, preset fast
  - `max` → CRF 14, preset medium
- `--codec h264`:
  - `low` → CRF 30, preset veryfast
  - `balanced` → CRF 23, preset veryfast
  - `high` → CRF 18, preset fast
  - `max` → CRF 15, preset medium

### Codec Fallback Behavior

- Default is `--codec h265`.
- If `libx265` is unavailable but `libx264` exists, Snapstream automatically falls back to H.264 and prints a warning.
- If you explicitly set `--codec h264`, no fallback is used; `libx264` must be available.
- If neither encoder exists, startup fails with a clear error.

## Output Layout

Chunks are written as:

```text
<directory>/YYYY-MM-DD/<timestamp_ms>_m<monitor_id>.mp4
```

Example:

```text
captures/2026-04-15/1771221600123_m1.mp4
```

## Examples

Capture monitor 1 at 5 FPS with 20-second chunks:

```bash
cargo run -- --monitor-id 1 --fps 5 --chunk-seconds 20 --directory ./captures
```

Capture exactly 120 frames:

```bash
cargo run -- --fps 2 --frames 120 --directory ./captures
```

Use custom ffmpeg binary:

```bash
cargo run -- --ffmpeg-path /usr/local/bin/ffmpeg --fps 2 --directory ./captures
```

Force H.264 codec:

```bash
cargo run -- --codec h264 --fps 2 --directory ./captures
```

## Public Repo Safety

This is a public repository. Keep it clean and secret-free.

- Never commit generated captures (`captures/`, `screenpipe-captures/`, `exports/`).
- Never commit `.env` or machine-local config.
- Never upload screenshots/videos containing sensitive information.

Recommended checks before push:

```bash
git status
git log --oneline --decorate -n 20
git grep -n "API_KEY\|TOKEN\|SECRET\|PASSWORD\|PRIVATE KEY"
```

## Troubleshooting

- **`ffmpeg` not found**
  - Install ffmpeg or provide `--ffmpeg-path`.
- **No monitors listed**
  - Ensure display is connected and active.
- **Permission issues / blank capture**
  - Grant screen recording permission in OS privacy settings.
- **Large files**
  - Lower `--fps`, increase `--chunk-seconds`, or use `--video-quality low`.

## Development

```bash
cargo fmt
cargo check
cargo clippy --all-targets -- -W clippy::all
```
