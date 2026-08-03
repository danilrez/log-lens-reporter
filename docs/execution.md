# Execution progress

`log-lens-reporter/run` provides a dependency-free stage runner for test and build processes. It can replace successful command noise with a loader and the formatted report. It keeps raw output available for failures.

## Configuration

Add the optional `execution` section to `loglensreporter.config.json`:

```json
{
  "execution": {
    "mode": "auto",
    "commandOutput": "on-failure",
    "reportOutput": "after-completion",
    "warnings": "summary",
    "logDirectory": ".loglensreporter"
  }
}
```

| Option          | Values                            | Default            | Description                                                     |
| --------------- | --------------------------------- | ------------------ | --------------------------------------------------------------- |
| `mode`          | `auto`, `progress`, `verbose`     | `auto`             | Uses a loader or streams the original command output.           |
| `commandOutput` | `always`, `on-failure`, `never`   | `on-failure`       | Controls when captured non-reporter command output is printed.  |
| `reportOutput`  | `after-completion`, `live`        | `after-completion` | Prints at the end or streams stable rows for full reports.      |
| `warnings`      | `always`, `summary`, `on-failure` | `summary`          | Prints warning lines, a linked summary, or only failure output. |
| `logDirectory`  | `string`                          | `.loglensreporter` | Stores details for warning summaries. Relative paths use `cwd`. |

`auto` selects an animated loader for an interactive terminal. It uses verbose streaming for CI and other non-TTY output. Explicit `progress` uses a static progress line when animation is unavailable. If a connected reporter knows the total test count and emits completed file results during the run, the loader shows progress such as `37% · 190/513 tests`. Commands without reporter progress keep the normal loader.

`reportOutput: "after-completion"` keeps the loader visible for the whole run and prints the complete formatted report after the child process exits. This default gives the cleanest output and avoids churn.

With `reportOutput: "live"` and `reportDetail: "full"`, the formatted header replaces the preparation loader as soon as a connected reporter starts. Each stable file row and its diagnostics are printed immediately. The summary is printed at completion.

With `reportDetail: "summary"` or `"status"`, the compact report remains buffered. The loader continues to show live test progress, and the report is printed after completion. Setup noise and unrelated command output remain captured based on `commandOutput`. If a runner cannot provide stable file-level results, its report remains buffered until those results are available.

In progress mode, the stage runner recognizes reporter protocol output from either child stdout or stderr. Custom reporter sinks can therefore route report output to either stream without losing live progress or the final report.

Live report output applies to progress mode. In `auto` mode, CI and redirected output still select verbose mode, so their output remains deterministic.

## Running a stage

```ts
import { runStage } from 'log-lens-reporter/run';

const result = await runStage({
  title: 'Running unit tests',
  preparation: {
    provider: 'Vitest',
    kind: 'UNIT',
  },
  command: 'pnpm',
  args: ['run', 'test:unit'],
});

process.exitCode = result.exitCode;
```

The stage runner does not use a shell. Pass the executable and arguments separately. It inherits the current process environment. Values in `env` override individual variables:

```ts
const result = await runStage({
  title: 'Running integration tests',
  command: 'pnpm',
  args: ['run', 'test:integration'],
  cwd: process.cwd(),
  env: {
    NODE_ENV: 'test',
  },
});
```

## Output behavior

In progress mode:

- `preparation` shows `Preparing the Vitest test environment`, then switches to `Running Vitest tests`;
- the entire loader text, including progress, is italic; the provider name is additionally bold;
- the active stage title uses the configured kind accent, while its percentage and test progress keep the terminal's default color;
- `preparationTitle` provides a plain fallback for stages without provider metadata;
- the loader shows the completed-test percentage when the reporter provides progress;
- successful command output is captured;
- `log-lens-reporter` output is printed based on `reportOutput`;
- technical output is shown based on `commandOutput`;
- warnings follow the configured warning policy;
- failed commands reveal their captured output unless `commandOutput` is `never`;
- `SIGINT` and `SIGTERM` are forwarded to the child process;
- the returned result preserves the child exit code and terminating signal.

Provider preparation and execution use the configured kind accent. Successful standalone stages use green, failures use red, and warnings use yellow. The root `color` option controls both execution and reporter colors. Exactly one empty line separates reports, stage transitions, and warning summaries.

With `warnings: "summary"`, hidden warning lines are saved to a stage-specific file:

```text
⚠ 2 warnings hidden · Details: .loglensreporter/warnings-running-unit-tests.log
```

`Details` is clickable in terminals that support OSC 8 links. Each stage overwrites its previous warning file, so the default directory does not grow with every run. Files are created only when warnings are hidden. Relative `logDirectory` values are resolved from the stage `cwd`. Absolute paths are used as provided. Add the configured directory to the consumer repository's ignore file. If the directory or file cannot be written, the warning lines are printed instead.

In verbose mode, stdout and stderr stream directly without a loader.

## Result

```ts
interface RunStageResult {
  exitCode: number;
  signal: NodeJS.Signals | null;
  durationMs: number;
  stdout: string;
  stderr: string;
  succeeded: boolean;
}
```

The library does not terminate the parent process. The caller must set `process.exitCode` or decide whether later stages should run.
