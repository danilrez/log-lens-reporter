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

| Option            | Type      | Default            | Description                                           |
| ----------------- | --------- | ------------------ | ----------------------------------------------------- |
| `kind`            | `string`  | `'UNIT'`           | Prefix and accent category for the run.               |
| `title`           | `string`  | `'MOCHA TEST RUN'` | Header title, limited to the shared text width.       |
| `showDescription` | `boolean` | `true`             | Shows generated provider, base path, and test counts. |

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

See the [configuration reference](configuration.md) for all shared options and defaults.
