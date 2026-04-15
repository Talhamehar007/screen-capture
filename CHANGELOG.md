# Changelog

All notable changes to this project are documented in this file.

The format is based on Keep a Changelog.

## [Unreleased]

### Added

- No unreleased changes yet.

## [0.1.1] - 2026-04-15

### Fixed

- ffmpeg chunk output now explicitly sets `-f mp4` when writing temporary `*.mp4.part` files.
- Prevented repeated chunk writer failures on ffmpeg versions that reject muxer inference for `.part` suffixes.

### Changed

- Added regression coverage for ffmpeg chunk argument construction to keep `-f mp4` behavior stable.

## [0.1.0] - 2026-04-15

### Added

- Initial standalone repository and CLI for monitor capture.
- Configurable capture FPS and output directory.
- Chunked ffmpeg video encoding with quality controls.
- `--codec` option with `h265`/`h264` support.
- Automatic fallback from H.265 (`libx265`) to H.264 (`libx264`) when H.265 is unavailable.
- Detailed project guide and public-repo safety guidance in `README.md`.
- GitHub issue templates for bug reports, feature requests, and security-safe reporting.

### Changed

- Project/package/CLI naming aligned to `screen-capture`.
- Output pipeline changed from image snapshots to ffmpeg-encoded chunked MP4 video files.
