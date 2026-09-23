# Node.js test runner integration

The Node.js adapter reads the built-in `node:test` event stream. It requires Node.js 20 or newer and does not depend on a third-party test framework.

Failed test details are grouped directly below their file's `FAILED` row.

## Basic setup

Run the test runner with the package subpath as its reporter:

```bash
node --test --test-reporter=log-lens-reporter/node
```

For a package script:

```json
{
  "scripts": {
    "test": "node --test --test-reporter=log-lens-reporter/node"
  }
}
```

The reporter groups leaf tests by file, maps failed events to `FAILED`, and maps `skip` and `todo` events to `SKIPPED`. Node.js keeps control of the final exit code.

## Project configuration

The Node.js CLI does not pass an options object, so the root `loglensreporter.config.json` is the simplest way to configure it:

```json
{
  "borderStyle": "double",
  "node": {
    "kind": "UNIT",
    "title": "NODE SERVICE TESTS",
    "borderStyle": "single"
  }
}
```

The `node` section overrides shared root values. A local wrapper is needed only when inline or function-valued options are required:

```js
// test/log-lens-reporter.cjs
const logLens = require('log-lens-reporter/node');

module.exports = (source) =>
  logLens(source, {
    title: 'NODE SERVICE TESTS',
    showDescription: true,
    kind: 'UNIT',
    color: true,
    border: false,
    borderStyle: 'single',
    links: {
      mode: 'auto',
    },
  });
```

Then point the CLI at the wrapper:

```bash
node --test --test-reporter=./test/log-lens-reporter.cjs
```

## Node-specific options

| Option               | Type      | Default                                 | Description                                                         |
| -------------------- | --------- | --------------------------------------- | ------------------------------------------------------------------- |
| `kind`               | `string`  | `'UNIT'`                                | Prefix and accent category for the run.                             |
| `title`              | `string`  | `'NODE TEST RUN'`                       | Header title, limited to the shared text width.                     |
| `showDescription`    | `boolean` | `true`                                  | Shows generated provider, base path, and test counts.               |
| `failureSummaryPath` | `string`  | `~/.loglensreporter/failure-summary.md` | Markdown report path; written for failed, flaky, or timed-out runs. |

If omitted, the report uses `~/.loglensreporter/failure-summary.md`. Configure `execution.logDirectory` to move the default file, pass an explicit path to choose another file, or use an empty string to disable it.

## Structured failure diagnostics

The adapter groups `node:test` attempts by test identity and file. Failed and flaky tests retain the error message, assertion `Expected`/`Received` values, call log, stack trace, and retry count when present in the event stream; structured details from every failed attempt are merged into the single diagnostic. A run containing only recovered retries is reported as `FLAKY`, not `PASSED`. `skip` and `todo` events are reported as `SKIPPED`; Node.js continues to own the process exit code.

See the [configuration reference](configuration.md) for all shared options and defaults.
