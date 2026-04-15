# screen-capture

`screen-capture` is a focused, standalone CLI for recording your screen into **ffmpeg-encoded MP4 chunks**.

It is intentionally limited to one responsibility: efficient screen capture.

## Getting Started in 60 Seconds

```bash
# 1) Ensure ffmpeg is installed
ffmpeg -version

# 2) List monitors
cargo run -- --list-monitors

# 3) Start recording (writes chunked MP4 files)
cargo run -- --fps 2 --directory ./captures
```

Stop with `Ctrl+C`.

First run note: your OS may ask for screen recording permission.

## Project Description

This project extracts only the capture pipeline from a broader recording stack and keeps it small, auditable, and practical for long-running local recording.

### In Scope

- monitor discovery and monitor selection
- all-monitor capture mode with periodic monitor re-scan
- capture-failure triggered monitor re-scan and writer restart
- periodic frame capture (`--fps`)
- ffmpeg-based chunked video encoding (`.mp4`)
- configurable codec, quality, and chunk duration

### Out of Scope

- OCR
- audio capture/transcription
- AI features
- servers, APIs, cloud sync, telemetry

## Why This Approach

Compared with writing individual images, chunked video provides:

- better compression and disk efficiency
- easier archival/transfer
- cleaner long-session recording
- lower filesystem overhead

## Scaffold Architecture

The project is intentionally split into small modules so capture behavior is easier to evolve and test:

- `src/main.rs` — entrypoint wiring
- `src/app.rs` — runtime loop, multi-monitor orchestration, chunk lifecycle
- `src/cli.rs` — CLI schema and runtime limits
- `src/monitor.rs` — monitor discovery/selection logic
- `src/capture.rs` — frame acquisition from monitor IDs
- `src/ffmpeg.rs` — ffmpeg discovery, encoder selection, codec fallback
- `src/writer.rs` — ffmpeg chunk writer and frame streaming

## Requirements

- Rust `1.92.0` (see `rust-toolchain.toml`)
- `ffmpeg` installed and available in PATH (or pass `--ffmpeg-path`)
- OS screen recording permission enabled

### Install ffmpeg

- macOS: `brew install ffmpeg`
- Ubuntu/Debian: `sudo apt install ffmpeg`
- Windows: install ffmpeg and ensure `ffmpeg.exe` is on PATH

## Build and Run

From project root:

```bash
cargo build --release
```

Binary path:

```text
target/release/screen-capture
```

Run with cargo:

```bash
cargo run -- --help
```

## Quick Start

List monitors:

```bash
cargo run -- --list-monitors
```

Start capture using defaults:

```bash
cargo run -- --fps 2 --directory ./captures
```

Stop with `Ctrl+C`.

## CLI Guide

```text
screen-capture [OPTIONS]
```

| Flag | Type | Default | Description |
|---|---|---|---|
| `--fps` | float | `2.0` | Capture rate (capped at 30 FPS) |
| `--directory` | path | `screenpipe-captures` | Output directory for video chunks |
| `--output-dir` | path | alias | Compatibility alias for `--directory` |
| `--monitor-id` | int (repeatable) | auto | Target monitor IDs |
| `--use-all-monitors` | bool | `true` | Capture all monitors when no `--monitor-id` is provided |
| `--chunk-seconds` | int | `30` | Rotate encoded chunk every N seconds |
| `--video-quality` | enum | `balanced` | `low`, `balanced`, `high`, `max` |
| `--codec` | enum | `h265` | `h265`, `h264` |
| `--frames` | int | unlimited | Capture exactly N frames, then exit |
| `--ffmpeg-path` | path | `ffmpeg` | ffmpeg binary path |
| `--monitor-rescan-seconds` | int | `5` | Monitor re-scan interval when capturing all monitors |
| `--capture-failure-rescan-threshold` | int | `3` | Force monitor re-scan after N consecutive capture failures |
| `--recover-partial-chunks` | bool | `true` | Attempt startup recovery of `*.mp4.part` leftovers |
| `--list-monitors` | bool | `false` | Print monitors and exit |

### Codec and Fallback Behavior

- Default codec: `h265` (`libx265`)
- If `libx265` is missing but `libx264` exists, `h265` requests auto-fallback to `h264` with a warning.
- If `--codec h264` is explicitly requested, `libx264` must exist.
- If neither encoder exists, startup fails with a clear error.

### Crash Recovery Behavior

- In-progress chunks are written as `*.mp4.part` and finalized to `*.mp4` on clean close.
- On startup, `screen-capture` scans for leftover `*.mp4.part` files and attempts best-effort remux recovery.
- Recovery summary is printed at startup (`scanned/recovered/failed/skipped`).

### Quality Mapping

`--video-quality` maps to codec-specific CRF/preset values:

- `--codec h265`
  - `low`: CRF 32, preset ultrafast
  - `balanced`: CRF 23, preset ultrafast
  - `high`: CRF 18, preset fast
  - `max`: CRF 14, preset medium
- `--codec h264`
  - `low`: CRF 30, preset veryfast
  - `balanced`: CRF 23, preset veryfast
  - `high`: CRF 18, preset fast
  - `max`: CRF 15, preset medium

## Output Format

Video chunks are written to:

```text
<directory>/YYYY-MM-DD/<timestamp_ms>_m<monitor_id>.mp4
```

Example:

```text
captures/2026-04-15/1771221600123_m1.mp4
```

## Usage Examples

Capture monitor `1` at 5 FPS, 20-second chunks:

```bash
cargo run -- --monitor-id 1 --fps 5 --chunk-seconds 20 --directory ./captures
```

Capture all monitors (default behavior):

```bash
cargo run -- --use-all-monitors true --fps 2 --directory ./captures
```

Capture only selected monitors:

```bash
cargo run -- --use-all-monitors false --monitor-id 1 --monitor-id 3 --fps 2 --directory ./captures
```

Force H.264:

```bash
cargo run -- --codec h264 --fps 2 --directory ./captures
```

Capture fixed number of frames:

```bash
cargo run -- --fps 2 --frames 120 --directory ./captures
```

Use custom ffmpeg binary:

```bash
cargo run -- --ffmpeg-path /usr/local/bin/ffmpeg --fps 2 --directory ./captures
```

## Operational Recommendations

For long-running capture:

- Start with `--fps 1` or `--fps 2`
- Use `--video-quality balanced` for good space/quality tradeoff
- Increase `--chunk-seconds` (for example 60) to reduce chunk file count
- Monitor free disk space regularly

## Privacy and Public Repo Safety

This repository is public. Keep generated data and secrets out of git.

- Do not commit capture outputs (`captures/`, `screenpipe-captures/`, `exports/`)
- Do not commit `.env`, API tokens, private keys, or machine-local files
- Treat all screen recordings as potentially sensitive

Recommended pre-push checks:

```bash
git status
git log --oneline --decorate -n 20
git grep -n "API_KEY\|TOKEN\|SECRET\|PASSWORD\|PRIVATE KEY"
```

## Troubleshooting

- **`ffmpeg` not found**
  - Install ffmpeg or pass `--ffmpeg-path`.
- **No monitors listed**
  - Verify monitor connection and run `--list-monitors`.
- **Blank/blocked capture**
  - Grant screen recording permission in OS privacy settings.
- **High disk usage**
  - Reduce `--fps`, lower quality, or increase chunk duration.

## Development

```bash
cargo fmt
cargo check
cargo clippy --all-targets -- -W clippy::all
```
