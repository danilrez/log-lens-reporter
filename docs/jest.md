# Jest integration

The Jest adapter supports JavaScript and TypeScript projects that use Jest 30. It prints one result row per test file, failure details, totals that include retries, and the shared `log-lens-reporter` summary.

Failed assertions and test-file errors are grouped directly below their file's `FAILED` row. Global run errors remain standalone diagnostics.

## Basic setup

Add the reporter to `jest.config.js`:

```js
module.exports = {
  reporters: ['log-lens-reporter/jest'],
};
```

This replaces Jest's default reporter. To keep both reporters, include `default` explicitly:

```js
module.exports = {
  reporters: ['default', 'log-lens-reporter/jest'],
};
```

Keeping both produces duplicate progress output. In most cases, use only the `log-lens-reporter` reporter.

## Configuration

Jest passes options through a reporter tuple:

```js
module.exports = {
  reporters: [
    [
      'log-lens-reporter/jest',
      {
        title: 'APPLICATION TESTS',
        showDescription: true,
        kind: 'UNIT',
        color: true,
        border: true,
        borderStyle: 'double',
        links: {
          mode: 'auto',
        },
      },
    ],
  ],
};
```

## Jest-specific options

| Option            | Type      | Default           | Description                                           |
| ----------------- | --------- | ----------------- | ----------------------------------------------------- |
| `kind`            | `string`  | `'UNIT'`          | Prefix and accent category for the run.               |
| `title`           | `string`  | `'JEST TEST RUN'` | Header title, limited to the shared text width.       |
| `showDescription` | `boolean` | `true`            | Shows generated provider, base path, and test counts. |

## Project configuration and overrides

The basic reporter setup automatically reads the root `loglensreporter.config.json`:

```json
{
  "borderStyle": "double",
  "jest": {
    "title": "APPLICATION TESTS",
    "borderStyle": "single"
  }
}
```

Jest uses `single`; other providers keep the root `double` value. Options in the Jest reporter tuple override both levels.

The adapter maps failed assertions to `FAILED`, pending/todo/disabled assertions to `SKIPPED`, and a passing assertion with multiple invocations to `FLAKY`. Jest still owns the process exit code.

See the [configuration reference](configuration.md) for all shared options and defaults.
