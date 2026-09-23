# Playwright integration

The Playwright adapter groups final results by source file and includes the duration of every retry. It distinguishes flaky and timed-out results and prints errors and attachment paths.

Failed and timed-out test details, including artifact paths, are grouped directly below their file result row. Global reporter errors remain standalone diagnostics.

## Install

```bash
pnpm add -D log-lens-reporter @playwright/test
```

## Minimal configuration

```ts
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  reporter: [['log-lens-reporter/playwright']],
});
```

## Full example

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  reporter: [
    [
      'log-lens-reporter/playwright',
      {
        title: 'PLAYWRIGHT TEST RUN',
        showDescription: true,
        primaryKind: 'E2E',
        color: true,
        border: true,
        borderStyle: 'double',
        ensureOutputDirectories: true,
        links: {
          mode: 'auto',
          scope: 'basename',
          workspaceRoot: process.cwd(),
        },
      },
    ],
  ],
});
```

## Playwright-specific options

| Option                    | Type                       | Default                                 | Description                                                         |
| ------------------------- | -------------------------- | --------------------------------------- | ------------------------------------------------------------------- |
| `title`                   | `string`                   | `'PLAYWRIGHT TEST RUN'`                 | Header title.                                                       |
| `showDescription`         | `boolean`                  | `true`                                  | Shows generated provider, base path, and test counts.               |
| `primaryKind`             | `string`                   | first test category                     | Accent used for the complete run rail and summary.                  |
| `classifyPath`            | `(path: string) => string` | built-in classifier                     | Selects a category from a source path.                              |
| `ensureOutputDirectories` | `boolean`                  | `true`                                  | Recreates Playwright output directories before completion.          |
| `failureSummaryPath`      | `string`                   | `~/.loglensreporter/failure-summary.md` | Markdown report path; written for failed, flaky, or timed-out runs. |

The generated base path is based on every file in the current run. Mixed-category runs show their nearest shared test root. `primaryKind` still controls the full-run rail and summary accent.

## Structured failure diagnostics

When a test fails across retries, attempts and attachments (`error-context.md`, `trace.zip`, screenshots) are aggregated into a single nested failure block with a retry counter `(× N retries)`. Error messages, assertion diffs, call logs, and stack frames are preserved when Playwright provides them. Global reporter errors remain standalone. By default, a Markdown summary is written to `~/.loglensreporter/failure-summary.md` when the run contains failed, flaky, or timed-out tests. Configure `execution.logDirectory` to move the default file, pass an explicit `failureSummaryPath` for another path, or use an empty string to disable it.

## Project configuration and overrides

The minimal reporter setup automatically reads the root `loglensreporter.config.json`:

```json
{
  "borderStyle": "double",
  "playwright": {
    "primaryKind": "E2E",
    "title": "E2E TEST RUN",
    "borderStyle": "single"
  }
}
```

Playwright uses `single`; other providers keep the root `double` value. Options in the reporter tuple override both levels.

## Default path classification

| Path segment    | Category      |
| --------------- | ------------- |
| `/integration/` | `INTEGRATION` |
| `/security/`    | `SECURITY`    |
| `/smoke/`       | `SMOKE`       |
| `/host/`        | `HOST`        |
| any other path  | `E2E`         |

Custom classifier:

```ts
{
  classifyPath: (filePath: string) => {
    if (filePath.includes('/api/')) return 'API';
    if (filePath.includes('/component/')) return 'COMPONENT';
    return 'E2E';
  },
  kindStyles: {
    API: {
      foreground: 'white',
      background: 'bgMagenta',
      accent: 'magenta',
    },
  },
}
```

See the [configuration reference](configuration.md) for shared options.
