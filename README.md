# snapstream

`snapstream` is a standalone, minimal screen capture CLI.

It does one job: capture monitor frames at a configurable rate and write JPEG snapshots to disk.

## Scope

Included:

- monitor discovery
- periodic screen capture (`--fps`)
- JPEG snapshot persistence (`--directory`)
- optional resize and quality controls

Explicitly excluded:

- OCR
- audio capture/transcription
- AI summarization/inference
- HTTP/WebSocket server APIs
- cloud sync

## Quick Start

From the `snapstream/` directory:

```bash
cargo run -- --list-monitors
```

```bash
cargo run -- --fps 2 --directory ./captures
```

Press `Ctrl+C` to stop.

## Build

```bash
cargo build
```

## CLI Usage

```bash
snapstream [OPTIONS]
```

### Primary flags

- `--fps <FLOAT>`
  - Capture rate in frames per second.
  - Default: `2.0`

- `--directory <PATH>`
  - Output directory for snapshots.
  - Default: `screenpipe-captures`

### Compatibility alias

- `--output-dir <PATH>`
  - Alias for `--directory`.

### Other flags

- `--monitor-id <ID>`
  - Capture a specific monitor.
  - If omitted, uses primary monitor or first available.

- `--jpeg-quality <1-100>`
  - JPEG quality setting.
  - Default: `80`

- `--max-width <PX>`
  - Maximum width of output image.
  - Default: `1920`
  - `0` disables resizing.

- `--list-monitors`
  - Print detected monitors and exit.

- `--frames <N>`
  - Capture exactly `N` frames and exit.

## Output Layout

Snapshots are written as:

```text
<directory>/YYYY-MM-DD/<timestamp_ms>_m<monitor_id>.jpg
```

Example:

```text
captures/2026-04-15/1771221600123_m1.jpg
```

## Example Commands

List monitors:

```bash
cargo run -- --list-monitors
```

Capture monitor 1 at 5 FPS:

```bash
cargo run -- --monitor-id 1 --fps 5 --directory ./captures
```

Capture 50 frames only:

```bash
cargo run -- --fps 2 --frames 50
```

## Runtime Notes

- On first run, your OS may request screen recording permission.
- If capture fails after sleep/wake or display reconfiguration, restart the process.
- If no monitor appears, run with `--list-monitors` and verify OS display state.

## Development

Format:

```bash
cargo fmt
```

Check:

```bash
cargo check
```

Clippy:

```bash
cargo clippy --all-targets -- -W clippy::all
```

