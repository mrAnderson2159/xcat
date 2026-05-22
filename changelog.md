# Changelog

## [0.1.1] - 2026-05-22

### Fixed

- Prevent `xcat` from accidentally loading `.env` files from scanned/current directories as application configuration. This ensures that environment variables defined in `.env` files do not interfere with `xcat`'s operation, providing a more predictable and stable user experience.

## 0.1.0

Initial standalone release of `xcat`.

### Added

- CLI entry point through `xcat`
- Recursive scanning of files and directories
- Support for multiple input paths
- Structured file headers in generated output
- Relative path headers based on the common display root
- Ignore patterns through `-I/--ignore`
- Built-in ignore presets
- `.gitignore`-style ignore file support
- File output through `-o/--output`
- Safe overwrite protection
- Explicit overwrite with `-f/--force`
- UTF-8 stdout configuration for better cross-platform behavior
