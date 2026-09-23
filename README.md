# log-lens-reporter

`log-lens-reporter` is a unified terminal reporter for clear, consistent, and actionable test output. It prints one progress row per file, groups failed test details directly below that row, and finishes with a shared summary table.

Public documentation and issue tracking are available in the [public repository](https://github.com/danilrez/log-lens-reporter). See the [changelog](https://github.com/danilrez/log-lens-reporter/blob/main/CHANGELOG.md) for release notes.

## Default output

The default presentation uses colors, Unicode double borders, category-specific prefixes, and clickable file names in supported terminals. The header, progress body, and summary use connected sections of equal width. They share borders and adapt to the terminal width.

```text
╔════════════════════════════════════════════════════════════╗
║  UNIT · VITEST TEST RUN                                    ║
║  Vitest · tests · 2 files · 14 tests                       ║
╟────────────────────────────────────────────────────────────╢
║   UNIT  tests/auth.spec.ts  ·  8 tests · 42ms · PASSED     ║
║   UNIT  tests/store.spec.ts ·  6 tests · 91ms · FAILED     ║
╟────────────────────────────────────────────────────────────╢
║  SUMMARY                                                   ║
╟┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈╢
║  Result: FAILED · Duration: 184ms · Passed: 13 · Failed: 1 ║
║  Timed out: 0 · Flaky: 0 · Skipped: 0                      ║
╚════════════════════════════════════════════════════════════╝
```

## Failure diagnostics

Failed file rows include one nested diagnostic block per failed or flaky test. The shared diagnostic model preserves the failure message, `Expected`/`Received` values (including explicit empty, whitespace-only, and `null` values), call log, stack trace, retry count, and runner attachments when the adapter provides them. Global runner and process errors remain standalone diagnostics instead of being attached to the last test file.

Playwright combines retry attempts and attachments for the same test into one block. Vitest, Jest, Mocha, Node.js, and Go adapters map their runner-specific assertion and process errors into the same format. Infrastructure output from `runStage` is captured and filtered in `auto`/`progress` mode; use `verbose` when raw child output is required.

Failed, flaky, and timed-out runs also produce a Markdown failure summary at `~/.loglensreporter/failure-summary.md`. Set `failureSummaryPath` to choose another path, set it to `''` to disable the artifact, or pass one `FailureSummaryCollector` to multiple `runStage` calls to combine their failures.

## Install

| Package manager | Command                                    |
| --------------- | ------------------------------------------ |
| npm             | `npm install --save-dev log-lens-reporter` |
| yarn            | `yarn add --dev log-lens-reporter`         |
| pnpm            | `pnpm add -D log-lens-reporter`            |

## Configuration

All adapters use the same presentation options:

| Option               | Accepted values                      | Default                                 | Description                                                  |
| -------------------- | ------------------------------------ | --------------------------------------- | ------------------------------------------------------------ |
| `color`              | `true`, `false`                      | `true`                                  | Disables regular ANSI colors; critical emphasis remains      |
| `border`             | `true`, `false`                      | `true`                                  | Enables or disables the Unicode frame                        |
| `borderStyle`        | `'double'`, `'single'`               | `'double'`                              | Produces `╔══╗` or `┌──┐` when borders are enabled           |
| `sectionWidth`       | `number`                             | terminal                                | Sets the shared inner width for all three blocks             |
| `showDescription`    | `boolean`                            | `true`                                  | Shows generated provider, base path, file, and test metadata |
| `terminalWidth`      | `number`                             | detected                                | Overrides terminal detection                                 |
| `terminalMargin`     | `number`                             | `4`                                     | Keeps output away from the terminal auto-wrap edge           |
| `pathWidth`          | `number`                             | available space                         | Optionally limits the path column before adaptive shortening |
| `links`              | `boolean`, `LinkConfig`              | automatic                               | Controls clickable file links                                |
| `kindStyles`         | `Record<string, KindStyle>`          | built-in palette                        | Customizes category colors                                   |
| `reportDetail`       | `'full'`, `'summary'`, or `'status'` | `'full'`                                | Selects the full report, summary block, or one status line   |
| `failureSummaryPath` | `string`                             | `~/.loglensreporter/failure-summary.md` | Markdown failure report path                                 |

See the [configuration reference](https://github.com/danilrez/log-lens-reporter/blob/main/docs/configuration.md) for complete types, color support, link settings, custom categories, and examples.

Projects with several runners can add one `loglensreporter.config.json` at the repository root. Root options apply to every adapter. Provider sections override the root options, and inline reporter options have the highest priority. See [project configuration](https://github.com/danilrez/log-lens-reporter/blob/main/docs/configuration.md#project-configuration) for the schema and discovery rules.

Failed tests, flaky tests, and timed-out tests are written to `~/.loglensreporter/failure-summary.md` by default. The default report follows `execution.logDirectory` when that directory is configured. Set `failureSummaryPath` to choose another path, or set it to an empty string to disable the report.

## Execution progress

The optional [`log-lens-reporter/run`](https://github.com/danilrez/log-lens-reporter/blob/main/docs/execution.md) entry point runs external test or build stages. It shows separate loader labels for preparation and execution. Full reports can be printed after completion or streamed one stable file row at a time. Compact summary and status reports remain buffered while the loader shows the live percentage and completed test count. Hidden warning summaries and Markdown failure reports use `~/.loglensreporter` by default and can be moved with `execution.logDirectory`.

## Runner integrations

The npm package requires Node.js 20 or newer.

| Runner      | Entry point                    | Requirement                             | Guide                                                                                           |
| ----------- | ------------------------------ | --------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Vitest      | `log-lens-reporter/vitest`     | Vitest 4                                | [Setup and options](https://github.com/danilrez/log-lens-reporter/blob/main/docs/vitest.md)     |
| Jest        | `log-lens-reporter/jest`       | Jest 30                                 | [Setup and options](https://github.com/danilrez/log-lens-reporter/blob/main/docs/jest.md)       |
| `node:test` | `log-lens-reporter/node`       | Node.js 20+                             | [Setup and options](https://github.com/danilrez/log-lens-reporter/blob/main/docs/node.md)       |
| Mocha       | `log-lens-reporter/mocha`      | Mocha 10+                               | [Setup and options](https://github.com/danilrez/log-lens-reporter/blob/main/docs/mocha.md)      |
| Playwright  | `log-lens-reporter/playwright` | Playwright 1.58-compatible reporter API | [Setup and options](https://github.com/danilrez/log-lens-reporter/blob/main/docs/playwright.md) |
| Go          | `log-lens-reporter/go`         | Go available on `PATH`                  | [Setup and options](https://github.com/danilrez/log-lens-reporter/blob/main/docs/go.md)         |
