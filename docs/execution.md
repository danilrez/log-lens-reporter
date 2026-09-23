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
    "warnings": "summary"
  }
}
```

| Option          | Values                            | Default              | Description                                                                            |
| --------------- | --------------------------------- | -------------------- | -------------------------------------------------------------------------------------- |
| `mode`          | `auto`, `progress`, `verbose`     | `auto`               | Uses filtered progress output or explicitly streams the command.                       |
| `commandOutput` | `always`, `on-failure`, `never`   | `on-failure`         | Controls when captured non-reporter command output is printed.                         |
| `reportOutput`  | `after-completion`, `live`        | `after-completion`   | Prints at the end or streams stable rows for full reports.                             |
| `warnings`      | `always`, `summary`, `on-failure` | `summary`            | Prints warning lines, a linked summary, or only failure output.                        |
| `logDirectory`  | `string`                          | `~/.loglensreporter` | Stores warning logs and the default Markdown failure report. Relative paths use `cwd`. |

`auto` selects the progress path for both interactive terminals and CI. Interactive terminals use an animated loader; redirected and CI output uses a deterministic progress line. This keeps infrastructure output captured and filtered instead of streaming Docker/BuildKit noise directly. Explicit `progress` has the same behavior. If a connected reporter knows the total test count and emits completed file results during the run, the loader shows progress such as `37% · 190/513 tests`. Commands without reporter progress keep the normal loader.

`runStage` also accepts the shared formatter options (`border`, `borderStyle`, `sectionWidth`, `pathWidth`, `links`, `kindStyles`, `reportDetail`, `failureSummaryPath`, and related options). In progress mode these options are propagated to a connected `log-lens-reporter` child, so its captured report uses the same presentation as the outer stage. Each `runStage` call is isolated by default. To combine multiple stages in one Markdown failure report, create one `FailureSummaryCollector` and pass it to every stage in that run; independent calls never inherit earlier failures.

`reportOutput: "after-completion"` keeps the loader visible for the whole run and prints the complete formatted report after the child process exits. This default gives the cleanest output and avoids churn.

With `reportOutput: "live"` and `reportDetail: "full"`, the formatted header replaces the preparation loader as soon as a connected reporter starts. Each stable file row and its diagnostics are printed immediately. The summary is printed at completion.

With `reportDetail: "summary"` or `"status"`, the compact report remains buffered. The loader continues to show live test progress, and the report is printed after completion. Setup noise and unrelated command output remain captured based on `commandOutput`. If a runner cannot provide stable file-level results, its report remains buffered until those results are available.

In progress mode, the stage runner recognizes reporter protocol output from either child stdout or stderr. Custom reporter sinks can therefore route report output to either stream without losing live progress or the final report.

Live report output applies to progress mode. `auto` also uses progress mode for CI and redirected output, so connected reporter output remains filtered and uses the formatter options propagated by the outer stage.

## Failure diagnostics

In `auto` and `progress` modes, `runStage` captures reporter protocol output from both child stdout and stderr. Structured file results and nested failure diagnostics are rendered through the same formatter as direct adapters. Standalone process, protocol, and global runner errors remain global diagnostics; they are not assigned to the last file seen.

Docker/BuildKit and container lifecycle noise is filtered from captured command output in progress mode. Set `mode: 'verbose'` when the complete raw child stream is needed for debugging. A failure-summary filesystem warning never replaces the child result or exit code.

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

For an explicit multi-stage failure report, reuse a collector within the run:

```ts
import { FailureSummaryCollector, runStage } from 'log-lens-reporter/run';

const failureSummaryCollector = new FailureSummaryCollector();
await runStage({ title: 'Unit tests', command: 'pnpm', args: ['test:unit'], failureSummaryCollector });
await runStage({ title: 'Integration tests', command: 'pnpm', args: ['test:integration'], failureSummaryCollector });
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

Provider preparation and execution use the configured kind accent. Successful standalone stages use green, failures use red, and warnings use yellow. With `color: false`, regular execution colors are disabled, while failed-stage messages remain red and warnings remain yellow. Exactly one empty line separates reports, stage transitions, and warning summaries.

With `warnings: "summary"`, hidden warning lines are saved to a stage-specific file in the configured log directory:

```text
⚠ 2 warnings hidden · Details: ~/.loglensreporter/warnings-running-unit-tests.log
```

`Details` is clickable in terminals that support OSC 8 links. Each stage overwrites its previous warning file, so the directory does not grow with every run. Files are created only when warnings are hidden. Relative `logDirectory` values are resolved from the stage `cwd`; absolute paths are used as provided. If a repository-local directory is configured, add it to the consumer repository's ignore file. If the directory or file cannot be written, the warning lines are printed instead.

### Markdown failure summary

`runStage` writes a Markdown failure report to `~/.loglensreporter/failure-summary.md` by default. The report is created when a stage contains failed, flaky, or timed-out tests. A standalone passing stage clears an older artifact at the same path so it cannot be mistaken for the current run. Use an explicit `failureSummaryPath` to select a different file, or set it to an empty string to disable the report. To combine multiple stages, pass the same `FailureSummaryCollector` to each call. If the summary directory or file cannot be written, `runStage` emits a concise warning and preserves the child exit code and result. When `execution.logDirectory` is configured, the default file is placed in that directory as `failure-summary.md`.

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
