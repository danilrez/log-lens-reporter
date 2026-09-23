# Changelog

All notable changes to `log-lens-reporter` are documented in this file.

## [1.0.1] - 2026-09-23

### Failure diagnostics

- Unified structured failure details across Vitest, Jest, Mocha, Node.js, Playwright, and Go, including assertion values, call logs, stack traces, retry counts, and attachments when provided by the runner.
- Added a Markdown summary for failed, flaky, and timed-out runs. It defaults to `~/.loglensreporter/failure-summary.md`; `failureSummaryPath` can change the path or disable the report, and the default follows `execution.logDirectory`.
- Improved handling of Go process and build failures, Mocha filtered or bailed runs, and Playwright retries and attachments.

### Execution output

- Suppressed Docker build and container lifecycle noise in `auto` and `progress` modes; `verbose` mode keeps raw child output available.

### Terminal formatting

- Standardized bold and color emphasis for status markers and labels, including when color output is disabled.

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
