# snapstream

`snapstream` is a standalone, minimal, **screen-capture-only** CLI.

It does exactly one thing:

1. choose a monitor
2. capture frames at `--fps`
3. write JPEG snapshots to `--directory`

No OCR. No audio. No AI. No server. No cloud sync.

## Why This Exists

This project is extracted from a larger recording stack to provide a small, public-friendly utility focused only on capture.

- predictable behavior
- narrow scope
- easy to audit
- safe to integrate into other pipelines

## Scope

Included:

- monitor discovery and listing
- monitor selection by ID
- periodic frame capture
- JPEG persistence with quality controls
- optional output resizing

Out of scope:

- OCR/text extraction
- speech/audio processing
- AI inference/summarization
- APIs, background services, cloud features
- analytics and telemetry

## Installation

### Prerequisites

- Rust toolchain `1.92.0` (see `rust-toolchain.toml`)

### Build from source

From the `snapstream/` directory:

```bash
cargo build --release
```

Binary path:

```text
target/release/snapstream
```

### Run without manual build

```bash
cargo run -- --help
```

## Quick Start

List monitors:

```bash
cargo run -- --list-monitors
```

Capture using defaults:

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
| `--fps` | float | `2.0` | Capture rate (frames per second) |
| `--directory` | path | `screenpipe-captures` | Snapshot output directory |
| `--output-dir` | path | alias | Compatibility alias for `--directory` |
| `--monitor-id` | integer | auto | Capture specific monitor ID |
| `--jpeg-quality` | `1..=100` | `80` | JPEG quality |
| `--max-width` | integer | `1920` | Resize output width (`0` disables resize) |
| `--list-monitors` | bool | `false` | Print monitors and exit |
| `--frames` | integer | unlimited | Capture exactly N frames and exit |

Monitor auto-selection behavior:

- uses primary monitor if available
- falls back to first available monitor

## Output Layout

Snapshots are written to:

```text
<directory>/YYYY-MM-DD/<timestamp_ms>_m<monitor_id>.jpg
```

Example:

```text
captures/2026-04-15/1771221600123_m1.jpg
```

## Examples

Capture monitor `1` at 5 FPS:

```bash
cargo run -- --monitor-id 1 --fps 5 --directory ./captures
```

Capture exactly 50 frames:

```bash
cargo run -- --fps 2 --frames 50
```

Use alias flag:

```bash
cargo run -- --fps 1 --output-dir ./captures
```

## Privacy & Security

`snapstream` itself does not send data anywhere. It captures locally and writes local files.

However, screen captures may contain highly sensitive information (messages, credentials, tokens, personal data).

Before publishing or sharing:

1. review captured images
2. delete sensitive frames
3. never commit capture output directories
4. run a final history + content check before push

Recommended pre-push checks:

```bash
git status
git log --oneline --decorate -n 20
git grep -n "API_KEY\|TOKEN\|SECRET\|PASSWORD\|PRIVATE KEY"
```

## Public Repo Safety Rules

- Do not commit runtime captures (`screenpipe-captures/`, `captures/`, exports).
- Do not commit local env files, credentials, or machine-specific configs.
- Keep example commands generic and secret-free.
- If sensitive data is ever committed, rotate credentials and rewrite history before publishing.

## Architecture (High Level)

Single-process loop:

1. Parse CLI options
2. Enumerate monitors (`xcap`)
3. Select target monitor
4. Capture frame each tick (`--fps`)
5. Optional resize (`--max-width`)
6. Encode JPEG (`--jpeg-quality`)
7. Persist to date-based directory

## Troubleshooting

- **No monitors found**
  - Verify display is connected and active.
  - Re-run `--list-monitors`.
- **Permission denied / black captures**
  - Grant screen recording permission in your OS privacy settings.
- **Capture fails after sleep/display changes**
  - Restart the process.
- **High CPU or disk usage**
  - Reduce `--fps`.
  - Lower `--max-width`.
  - Lower `--jpeg-quality`.

## Development

Format:

```bash
cargo fmt
```

Type-check/build:

```bash
cargo check
```

Lint:

```bash
cargo clippy --all-targets -- -W clippy::all
```

