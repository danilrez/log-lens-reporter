# Vitest integration

The Vitest adapter supports the Vitest 4 reporter lifecycle. It groups completed test cases into one row per module, reports successful retries as flaky, keeps failure details, and prints one final summary.

Failed test details are grouped directly below their module's `FAILED` row. Unhandled run errors remain standalone diagnostics.

## Install

```bash
pnpm add -D log-lens-reporter vitest
```

## Minimal configuration

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    reporters: ['log-lens-reporter/vitest'],
  },
});
```

Do not add Vitest's `default` reporter unless duplicate terminal progress is intentional.

## Full example

```ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    reporters: [
      [
        'log-lens-reporter/vitest',
        {
          kind: 'UNIT',
          title: 'VITEST TEST RUN',
          showDescription: true,
          color: true,
          border: true,
          borderStyle: 'double',
          links: {
            mode: 'auto',
            scope: 'basename',
            workspaceRoot: process.cwd(),
          },
        },
      ],
    ],
  },
});
```

## Vitest-specific options

| Option               | Type      | Default                                 | Description                                                         |
| -------------------- | --------- | --------------------------------------- | ------------------------------------------------------------------- |
| `kind`               | `string`  | `'UNIT'`                                | Prefix and accent category for the run.                             |
| `title`              | `string`  | `'VITEST TEST RUN'`                     | Header title, limited to the shared text width.                     |
| `showDescription`    | `boolean` | `true`                                  | Shows generated provider, base path, and test counts.               |
| `failureSummaryPath` | `string`  | `~/.loglensreporter/failure-summary.md` | Markdown report path; written for failed, flaky, or timed-out runs. |

If omitted, the report uses `~/.loglensreporter/failure-summary.md`. Configure `execution.logDirectory` to move the default file, pass an explicit path to choose another file, or use an empty string to disable it.

## Structured failure diagnostics

Failed and flaky test cases are rendered below their module row with the test title, retry count, normalized failure message, assertion `Expected`/`Received` values, call log, and stack trace when Vitest provides them. Module errors and unhandled run errors remain standalone diagnostics. `todo`, `pending`, and skipped cases are reported as `SKIPPED` and are not treated as failures.

## Project configuration and overrides

The minimal reporter setup automatically reads the root `loglensreporter.config.json`:

```json
{
  "borderStyle": "double",
  "vitest": {
    "kind": "UNIT",
    "title": "UNIT TEST RUN",
    "borderStyle": "single"
  }
}
```

Vitest uses `single`; other providers keep the root `double` value. Options in the reporter tuple override both levels.

## Custom category

```ts
{
  kind: 'CONTRACT',
  title: 'CONTRACT TEST RUN',
  kindStyles: {
    CONTRACT: {
      foreground: 'white',
      background: 'bgBlue',
      accent: 'blue',
    },
  },
}
```

See the [configuration reference](configuration.md) for colors, borders, links, and sinks.
