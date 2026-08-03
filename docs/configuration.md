# Configuration reference

All adapters accept the shared formatter options. Runner adapters add their own lifecycle and execution options.

## Shared options

| Option            | Type                              | Default          | Description                                                                |
| ----------------- | --------------------------------- | ---------------- | -------------------------------------------------------------------------- |
| `color`           | `boolean`                         | `true`           | Enables or disables reporter and execution-stage ANSI colors.              |
| `border`          | `boolean`                         | `true`           | Enables or disables the Unicode frame.                                     |
| `borderStyle`     | `'double' \| 'single'`            | `'double'`       | Selects double or single border weight.                                    |
| `sectionWidth`    | `number`                          | terminal         | Sets the shared inner width for the header, body, and footer sections.     |
| `showDescription` | `boolean`                         | `true`           | Shows generated provider, base path, file, and test metadata.              |
| `terminalWidth`   | `number`                          | detected         | Overrides width detection for redirected output or deterministic tests.    |
| `terminalMargin`  | `number`                          | `4`              | Reserves columns at the right edge to prevent terminal auto-wrap.          |
| `pathWidth`       | `number`                          | available space  | Optionally limits the path column before adaptive shortening.              |
| `links`           | `boolean \| LinkConfig`           | automatic        | Controls OSC 8 file links.                                                 |
| `kindStyles`      | `Record<string, KindStyle>`       | built-in palette | Adds or overrides category colors.                                         |
| `reportDetail`    | `'full' \| 'summary' \| 'status'` | `'full'`         | Controls report detail level (full report, summary block, or status line). |
| `sink`            | `OutputSink`                      | `console`        | Replaces `console.log` and `console.error` for adapters.                   |

### Default formatter options

```ts
{
  color: true,
  border: true,
  borderStyle: 'double',
  showDescription: true,
  reportDetail: 'full',
  terminalMargin: 4,
  links: {
    mode: 'auto',
    scope: 'basename',
    workspaceRoot: process.cwd(),
  },
}
```

### Report detail

Use `reportDetail` to reduce successful run output while keeping the same category styling:

| Value     | Output                                                                                 |
| --------- | -------------------------------------------------------------------------------------- |
| `full`    | Header, per-file rows, diagnostics, and summary. This is the default.                  |
| `summary` | A framed summary block with the category badge, result, duration, and counters.        |
| `status`  | One unframed line with the run title on the category background, result, and duration. |

Compact modes still print failed, timed-out, and interrupted file rows with their diagnostics before the final summary or status line. Passing file rows remain hidden.

The `SUMMARY` heading uses the active category accent color in every detail and border mode.

When `border` is `false`, `full` mode adds a left-aligned rail to each file row, while compact `summary` mode adds the rail to each summary values row. The default `double` style uses `▎`; `borderStyle: 'single'` uses `▏`. Framed output is unchanged.

### Project configuration

Create an optional `loglensreporter.config.json` at the repository root to configure every adapter in one place:

```json
{
  "color": true,
  "border": true,
  "borderStyle": "double",
  "execution": {
    "mode": "auto",
    "commandOutput": "on-failure",
    "reportOutput": "after-completion",
    "warnings": "summary",
    "logDirectory": ".loglensreporter"
  },
  "vitest": {
    "kind": "UNIT",
    "showDescription": true,
    "borderStyle": "single"
  },
  "playwright": {
    "primaryKind": "E2E",
    "color": false
  },
  "go": {
    "directory": "backend"
  }
}
```

Options at the root apply to every adapter. A provider section overrides the matching root values only for that provider. Options passed directly to a reporter remain the highest priority:

```text
package defaults → root config → provider config → inline reporter options
```

For example, this inline Vitest option overrides both root and `vitest` values:

```ts
reporters: [['log-lens-reporter/vitest', { borderStyle: 'double' }]];
```

Configuration discovery starts in the runner working directory and moves upward until it finds the first `loglensreporter.config.json`. If no file exists, adapters use their built-in defaults. Invalid JSON, unknown keys, and invalid values produce an error with the full config path and option name.

Nested `links`, `kindStyles`, and Go `env` objects merge across the same precedence levels. JSON configuration cannot contain function-valued options: `sink` and Playwright `classifyPath` remain inline-only.

Provider-specific JSON options:

| Provider     | Additional options                                                        |
| ------------ | ------------------------------------------------------------------------- |
| `vitest`     | `kind`, `title`                                                           |
| `jest`       | `kind`, `title`                                                           |
| `node`       | `kind`, `title`                                                           |
| `mocha`      | `kind`, `title`                                                           |
| `playwright` | `title`, `primaryKind`, `ensureOutputDirectories`                         |
| `go`         | `cwd`, `directory`, `packages`, `goBinary`, `goArguments`, `env`, `title` |

All provider sections also accept the shared formatter options from the table above.

The formatter reads the width from `stdout`, `stderr`, or `COLUMNS`. Automatically detected widths are capped at `120` columns to keep output readable on wide terminals. A numeric `terminalWidth` bypasses this cap. By default, the formatter reserves four columns, even when `terminalWidth` is set. Terminal wrappers and automatic line wrapping may use part of the reported width. Set `terminalMargin: 0` to remove the reserve, or increase it when another process adds a prefix to every output line.

When no width is available, the formatter uses an `80`-column fallback and renders at most `76` visible columns. The header, progress body, and summary are connected sections with the same outer width. Neighboring sections share one border, so empty rows or duplicate border rows do not separate them.

The summary places the result, duration, and all counters on one row when they fit. A dotted divider separates the title from the values. On narrower terminals, complete value segments wrap without being clipped. Progress rows and diagnostics are padded to the shared right border.

By default, test paths use all available space in the path column. When a path does not fit, parent directories are shortened from left to right by only the required number of characters. The file name is shortened only after every parent directory reaches one character. An explicit `pathWidth` limits the column. Every adapter uses the same layout and path rules. Long diagnostics wrap inside the body frame instead of relying on terminal auto-wrap.

The active width is captured when a run header is printed. The body, diagnostics, and summary reuse this width. This prevents a terminal resize during a streaming test run from splitting the right border across different columns.

Divider weight is consistent across border styles. Section block dividers (between header, body, and footer) always use a solid `─` line regardless of `borderStyle`. The summary title–values separator always uses a dashed `┈` line regardless of `borderStyle`.

Runner headers limit title and metadata text to `48` visible columns. Metadata uses the format `Provider · base/path · N files · N tests`. Adapters find the shared base path from all collected test files, including every category in a mixed Playwright run. Set `showDescription: false` to show only the title row.

Progress rows reserve stable columns for the test path, test count, duration, and final status. Directory segments are shortened when needed. `PASSED`, `FAILED`, `FLAKY`, and the other outcomes remain aligned. Shortened file names use the ASCII marker `...` because a Unicode ellipsis can have an unclear terminal width. On narrow terminals, the formatter first removes alignment padding and then shortens the test-count label. It clips other content only when needed, so the right border stays inside the active width.

## Borders

Unicode double:

```text
╔════════╗
║  RUN   ║
╚════════╝
```

Unicode single:

```text
┌────────┐
│  RUN   │
└────────┘
```

Disabled:

```text
RUN
```

Examples:

```ts
const singleUnicode = {
  border: true,
  borderStyle: 'single',
} as const;

const withoutFrame = { border: false } as const;
```

The same style is applied to headers, progress rails, diagnostics, and summary tables.

## Color policy

Color is an explicit switch. The default is `true`; set it to `false` when plain output is required:

```ts
const colorsOn = { color: true } as const;
const colorsOff = { color: false } as const;
```

Built-in status colors:

| Status        | Color  |
| ------------- | ------ |
| `PASSED`      | green  |
| `FAILED`      | red    |
| `SKIPPED`     | blue   |
| `FLAKY`       | cyan   |
| `TIMED OUT`   | yellow |
| `INTERRUPTED` | dimmed |

Built-in category styles:

| Category      | Prefix background | Accent         |
| ------------- | ----------------- | -------------- |
| `UNIT`        | green             | green          |
| `GO BACKEND`  | cyan              | cyan           |
| `E2E`         | blue              | blue           |
| `INTEGRATION` | magenta           | magenta        |
| `SECURITY`    | yellow            | yellow         |
| `SMOKE`       | bright black      | bright black   |
| `HOST`        | bright blue       | bright blue    |
| `COVERAGE`    | bright magenta    | bright magenta |

Unknown categories use the `UNIT` style unless overridden:

```ts
{
  kindStyles: {
    API: {
      foreground: 'white',
      background: 'bgMagenta',
      accent: 'magenta',
    },
  },
}
```

Styling supports the standard 16-color ANSI palette for foregrounds and backgrounds, together with bold, dimmed, and italic text. Use the exported `AnsiForegroundColor`, `AnsiBackgroundColor`, and `TextStyle` types for the available values. Exact shades depend on the terminal theme.

## File links

| Option              | Type                            | Default         | Description                                            |
| ------------------- | ------------------------------- | --------------- | ------------------------------------------------------ |
| `mode`              | `'auto' \| 'always' \| 'never'` | `'auto'`        | Controls OSC 8 link generation.                        |
| `scope`             | `'basename' \| 'path'`          | `'basename'`    | Links only the file name or the complete visible path. |
| `workspaceRoot`     | `string`                        | `process.cwd()` | Root used to create visible relative paths.            |
| `hostWorkspaceRoot` | `string`                        | unset           | Maps container-relative paths to a host checkout.      |

Docker example:

```ts
{
  links: {
    mode: 'always',
    scope: 'basename',
    workspaceRoot: '/app',
    hostWorkspaceRoot: '/Users/developer/project',
  },
}
```

The visible value stays `tests/example.spec.ts`, while the link target points to the host file.

## Output sink

Adapters write progress and its nested test diagnostics to `log`. This keeps their order in redirected output. Standalone process, build, and global diagnostics are written to `error`:

```ts
const captured: string[] = [];

const options = {
  sink: {
    log: (message: string) => captured.push(message),
    error: (message: string) => captured.push(message),
  },
};
```

This is useful for tests, IDE integrations, and custom log storage.

## Current limits

- Status colors and summary labels are not configurable.
- Custom colors are limited to the named ANSI palette; RGB and hex values are not supported.
- Duration formatting is fixed to rounded milliseconds below one second and one decimal second at or above one second.
