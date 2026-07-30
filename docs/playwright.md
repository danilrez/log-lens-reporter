# Playwright integration

The Playwright adapter groups final results by source file, includes all retry durations, distinguishes flaky and timed-out outcomes, and prints errors and attachment paths.

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

| Option                    | Type                       | Default                 | Description                                                |
| ------------------------- | -------------------------- | ----------------------- | ---------------------------------------------------------- |
| `title`                   | `string`                   | `'PLAYWRIGHT TEST RUN'` | Header title.                                              |
| `showDescription`         | `boolean`                  | `true`                  | Shows generated provider, base path, and test counts.      |
| `primaryKind`             | `string`                   | first test category     | Accent used for the complete run rail and summary.         |
| `classifyPath`            | `(path: string) => string` | built-in classifier     | Selects a category from a source path.                     |
| `ensureOutputDirectories` | `boolean`                  | `true`                  | Recreates Playwright output directories before completion. |

The generated base path is inferred from every file in the current run. Mixed-category runs therefore report their nearest shared test root, while `primaryKind` continues to control the complete run rail and summary accent.

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
