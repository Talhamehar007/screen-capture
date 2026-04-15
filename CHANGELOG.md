# Changelog

All notable changes to this project are documented in this file.

The format is based on Keep a Changelog.

## [Unreleased]

### Added

- `--codec` option with `h265`/`h264` support.
- Automatic fallback from H.265 (`libx265`) to H.264 (`libx264`) when H.265 is unavailable.
- Detailed project guide and public-repo safety guidance in `README.md`.
- GitHub issue templates for bug reports, feature requests, and security-safe reporting.

### Changed

- Project/package/CLI naming aligned to `screen-capture`.
- Output pipeline changed from image snapshots to ffmpeg-encoded chunked MP4 video files.

## [0.1.0]

### Added

- Initial standalone repository and CLI for monitor capture.
- Configurable capture FPS and output directory.
- Chunked ffmpeg video encoding with quality controls.

