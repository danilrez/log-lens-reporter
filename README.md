# log-lens-reporter

`log-lens-reporter` is a unified terminal reporter for clear, consistent, and actionable test output. It prints one progress row per file, groups failed test details directly below that row, and finishes with a shared summary table.

Public documentation and issue tracking are available in the [public repository](https://github.com/danilrez/log-lens-reporter). See the [changelog](https://github.com/danilrez/log-lens-reporter/blob/main/CHANGELOG.md) for release notes.

## Default output

The default presentation uses colors, Unicode double borders, category-specific prefixes, and clickable file basenames in supported terminals. The header, progress body, and summary are rendered as connected equal-width sections that share borders and adapt to the terminal.

```text
╔════════════════════════════════════════════════════════════╗
║  UNIT · VITEST TEST RUN                                    ║
║  Vitest · tests · 2 files · 14 tests                       ║
╟────────────────────────────────────────────────────────────╢
║   UNIT  tests/auth.spec.ts  ·  8 tests · 42ms · PASSED     ║
║   UNIT  tests/store.spec.ts ·  6 tests · 91ms · FAILED     ║
╟────────────────────────────────────────────────────────────╢
║  TEST RUN SUMMARY                                          ║
╟┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈╢
║  Result: FAILED · Duration: 184ms · Passed: 13 · Failed: 1 ║
║  Timed out: 0 · Flaky: 0 · Skipped: 0                      ║
╚════════════════════════════════════════════════════════════╝
```

## Install

| Package manager | Latest                                     | Beta                                            |
| --------------- | ------------------------------------------ | ----------------------------------------------- |
| npm             | `npm install --save-dev log-lens-reporter` | `npm install --save-dev log-lens-reporter@beta` |
| Yarn            | `yarn add --dev log-lens-reporter`         | `yarn add --dev log-lens-reporter@beta`         |
| pnpm            | `pnpm add -D log-lens-reporter`            | `pnpm add -D log-lens-reporter@beta`            |

## Configuration

All adapters use the same presentation options:

| Option            | Accepted values             | Default          | Description                                                  |
| ----------------- | --------------------------- | ---------------- | ------------------------------------------------------------ |
| `color`           | `true`, `false`             | `true`           | Enables or disables ANSI colors                              |
| `border`          | `true`, `false`             | `true`           | Enables or disables the Unicode frame                        |
| `borderStyle`     | `'double'`, `'single'`      | `'double'`       | Produces `╔══╗` or `┌──┐` when borders are enabled           |
| `sectionWidth`    | `number`                    | terminal         | Sets the shared inner width for all three blocks             |
| `showDescription` | `boolean`                   | `true`           | Shows generated provider, base path, file, and test metadata |
| `terminalWidth`   | `number`                    | detected         | Overrides terminal detection                                 |
| `terminalMargin`  | `number`                    | `4`              | Keeps output away from the terminal auto-wrap edge           |
| `pathWidth`       | `number`                    | available space  | Optionally limits the path column before adaptive shortening |
| `links`           | `boolean`, `LinkConfig`     | automatic        | Controls clickable file links                                |
| `kindStyles`      | `Record<string, KindStyle>` | built-in palette | Customizes category colors                                   |

See the [configuration reference](https://github.com/danilrez/log-lens-reporter/blob/main/docs/configuration.md) for complete types, color support, link settings, custom categories, and examples.

Projects with several runners can add one `loglensreporter.config.json` at the repository root. Root options apply to every adapter, provider sections override them, and inline reporter options have the highest priority. See [project configuration](https://github.com/danilrez/log-lens-reporter/blob/main/docs/configuration.md#project-configuration) for the schema and discovery rules.

## Execution progress

The optional [`log-lens-reporter/run`](https://github.com/danilrez/log-lens-reporter/blob/main/docs/execution.md) entry point runs external test or build stages with separate preparation and execution loader labels. When a connected reporter knows the test count, the loader also shows a live percentage and completed-test count. Hidden warning summaries link to details in a configurable local log directory.

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

## License

MIT
