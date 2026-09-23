# Mocha integration

The Mocha adapter supports JavaScript and TypeScript projects that use Mocha 10 or newer. It prints one progress row per file, keeps final failure details, and marks a passing retried test as `FLAKY`.

Final failure details are grouped directly below their file's `FAILED` row.

## Basic setup

Add the reporter to `.mocharc.cjs`:

```js
module.exports = {
  reporter: 'log-lens-reporter/mocha',
};
```

Or select it from the command line:

```bash
mocha --reporter log-lens-reporter/mocha
```

## Configuration

Use `reporterOptions` in the JavaScript configuration file so values keep their correct boolean and number types:

```js
module.exports = {
  reporter: 'log-lens-reporter/mocha',
  reporterOptions: {
    title: 'SERVICE TESTS',
    showDescription: true,
    kind: 'UNIT',
    color: true,
    border: true,
    borderStyle: 'single',
    links: {
      mode: 'auto',
    },
  },
};
```

## Mocha-specific options

| Option               | Type      | Default                                 | Description                                                         |
| -------------------- | --------- | --------------------------------------- | ------------------------------------------------------------------- |
| `kind`               | `string`  | `'UNIT'`                                | Prefix and accent category for the run.                             |
| `title`              | `string`  | `'MOCHA TEST RUN'`                      | Header title, limited to the shared text width.                     |
| `showDescription`    | `boolean` | `true`                                  | Shows generated provider, base path, and test counts.               |
| `failureSummaryPath` | `string`  | `~/.loglensreporter/failure-summary.md` | Markdown report path; written for failed, flaky, or timed-out runs. |

If omitted, the report uses `~/.loglensreporter/failure-summary.md`. Configure `execution.logDirectory` to move the default file, pass an explicit path to choose another file, or use an empty string to disable it.

## Project configuration and overrides

The basic reporter setup automatically reads the root `loglensreporter.config.json`:

```json
{
  "borderStyle": "double",
  "mocha": {
    "title": "SERVICE TESTS",
    "borderStyle": "single"
  }
}
```

Mocha uses `single`; other providers keep the root `double` value. Values in `reporterOptions` override both levels.

Passing tests map to `PASSED`, pending tests to `SKIPPED`, final failures to `FAILED`, and successful retries to `FLAKY`. Mocha still owns the process exit code.

## Structured failure diagnostics

Mocha failure blocks preserve the test title, retry count, error message, `expected`/`actual` values, and stack trace when supplied by the runner. Test identity is tracked per test object, so duplicate `fullTitle()` values do not collapse into one result. Filtered tests are excluded from file totals; a `--bail` run that stops before all selected tests is reported as `INTERRUPTED` with an incomplete-run diagnostic.

See the [configuration reference](configuration.md) for all shared options and defaults.
