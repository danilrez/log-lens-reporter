# Execution progress

`log-lens-reporter/run` provides a dependency-free process stage runner for test and build orchestration. It can replace successful command noise with a loader followed by the formatted reporter output, while preserving raw output for failures.

## Configuration

Add the optional `execution` section to `loglensreporter.config.json`:

```json
{
  "execution": {
    "mode": "auto",
    "commandOutput": "on-failure",
    "warnings": "summary",
    "logDirectory": ".loglensreporter"
  }
}
```

| Option          | Values                            | Default            | Description                                                     |
| --------------- | --------------------------------- | ------------------ | --------------------------------------------------------------- |
| `mode`          | `auto`, `progress`, `verbose`     | `auto`             | Uses a loader or streams the original command output.           |
| `commandOutput` | `always`, `on-failure`, `never`   | `on-failure`       | Controls when captured non-reporter command output is printed.  |
| `warnings`      | `always`, `summary`, `on-failure` | `summary`          | Prints warning lines, a linked summary, or only failure output. |
| `logDirectory`  | `string`                          | `.loglensreporter` | Stores details for warning summaries. Relative paths use `cwd`. |

`auto` selects an animated loader for an interactive terminal and verbose streaming for CI and other non-TTY output. Explicit `progress` uses a static progress line when animation is unavailable. When a connected reporter knows the total test count and emits completed file results during the run, the interactive loader includes live progress such as `37% · 190/513 tests`. Commands without reporter progress keep the normal loader.

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

The stage runner does not use a shell. Pass the executable and arguments separately. The current process environment is inherited, and `env` values override individual variables:

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
- `preparationTitle` remains available as a plain fallback for stages without provider metadata;
- the loader shows the completed-test percentage when the reporter provides progress;
- successful command output is captured;
- `log-lens-reporter` output is printed after the stage completes;
- technical output is shown according to `commandOutput`;
- warnings follow the configured warning policy;
- failed commands reveal their captured output unless `commandOutput` is `never`;
- `SIGINT` and `SIGTERM` are forwarded to the child process;
- the returned result preserves the child exit code and terminating signal.

Provider preparation and execution use the configured kind accent, successful standalone stages use green, failures use red, and warning output uses yellow. The root `color` option controls these execution colors together with reporter colors. Reports, stage transitions, and warning summaries are separated by an empty line.

With `warnings: "summary"`, hidden warning lines are saved to a stage-specific file:

```text
⚠ 2 warnings hidden · Details: .loglensreporter/warnings-running-unit-tests.log
```

`Details` is clickable in terminals that support OSC 8 links. Each stage overwrites its previous warning file, so the default directory does not grow on every run. Files are created only when warnings are hidden. Relative `logDirectory` values are resolved from the stage `cwd`; absolute paths are preserved. Add the configured directory to the consumer repository's ignore file. If the directory or file cannot be written, the warning lines are printed instead of being lost.

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

The library does not terminate the parent process. The caller remains responsible for assigning `process.exitCode` or deciding whether later stages should run.
