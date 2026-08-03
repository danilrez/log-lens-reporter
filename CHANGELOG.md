# Changelog

All notable changes to `log-lens-reporter` are documented in this file.

## [1.0.0] - 2026-08-03

Version 1.0.0 is the first stable release of `log-lens-reporter` and starts the official 1.x release line.

### Compact report detail

- Added `full`, `summary`, and `status` report detail modes for every supported test runner.
- Reduced successful output while keeping failure, timeout, and interruption details.
- Improved compact layouts, responsive widths, and spacing.

### Live report output

- Added `after-completion` and `live` report output modes, with `after-completion` as the default.
- Streamed full reports during a run. Compact reports keep the loader and print after completion.
- Added test counts and percentages to the loader when the test runner provides progress.
- Preserved warnings, failure details, signals, exit codes, and output from custom stdout or stderr sinks.

## [0.1.1] - 2026-07-30

### Documentation

- Added setup guides for Vitest, Jest, `node:test`, Mocha, Playwright, and Go.
- Added practical examples to the configuration and execution guides.
- Updated package links to use the public documentation repository.

This is a documentation-only maintenance release. Runtime behavior is unchanged.
