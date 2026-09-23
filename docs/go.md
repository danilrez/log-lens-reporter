# Go integration

The Go adapter runs `go test -json` and maps top-level tests and subtests to source files. It keeps package build failures, forwards termination signals, and returns the child process exit details.

Failed test details are grouped directly below their `FAILED` file row. Package and build failures without a source test remain standalone diagnostics; a package-level diagnostic is also retained when that package has a failed test. If Go exits with an ordinary non-zero code without structured events or stderr, the adapter creates a process-level failure so the totals and Markdown summary still describe the failure.

## TypeScript script

```ts
// scripts/test-go.ts
import { runGoTests } from 'log-lens-reporter/go';

const result = await runGoTests({
  directory: 'backend',
  packages: ['./...'],
  goArguments: ['-count=1'],
  title: 'NATIVE GO TEST RUN',
  showDescription: true,
  color: true,
  border: true,
  borderStyle: 'double',
});

process.exitCode = result.exitCode;
```

Run the script with a TypeScript-capable Node.js setup or compile it with the application.

## CommonJS script

```js
// scripts/test-go.cjs
const { runGoTests } = require('log-lens-reporter/go');

runGoTests({ directory: 'backend' })
  .then((result) => {
    process.exitCode = result.exitCode;
  })
  .catch((error) => {
    console.error(error);
    process.exitCode = 1;
  });
```

```json
{
  "scripts": {
    "test:go": "node scripts/test-go.cjs"
  }
}
```

## Go-specific options

| Option               | Type                | Default                                 | Description                                                         |
| -------------------- | ------------------- | --------------------------------------- | ------------------------------------------------------------------- |
| `cwd`                | `string`            | `process.cwd()`                         | Workspace root used by the process and path mapping.                |
| `directory`          | `string`            | `'backend'`                             | Go module directory passed to `go -C`.                              |
| `packages`           | `string[]`          | `['./...']`                             | Package patterns passed to `go test`.                               |
| `goBinary`           | `string`            | `'go'`                                  | Go executable name or absolute path.                                |
| `goArguments`        | `string[]`          | `[]`                                    | Extra flags inserted before package patterns.                       |
| `env`                | `NodeJS.ProcessEnv` | inherited                               | Environment overrides merged with `process.env`.                    |
| `title`              | `string`            | `'NATIVE GO TEST RUN'`                  | Header title.                                                       |
| `showDescription`    | `boolean`           | `true`                                  | Shows generated provider, base path, and test counts.               |
| `failureSummaryPath` | `string`            | `~/.loglensreporter/failure-summary.md` | Markdown report path; written for failed, flaky, or timed-out runs. |

If omitted, the report uses `~/.loglensreporter/failure-summary.md`. Configure `execution.logDirectory` to move the default file, pass an explicit path to choose another file, or use an empty string to disable it.

The generated header metadata reflects the tests selected by `packages`, `-run`, and `-skip`. For example, a run limited to `./internal/store` uses that directory as its base path instead of the whole Go module.

## Structured failure diagnostics

The adapter parses `go test -json` output and groups subtest names and non-structural `output` lines under the mapped source file. File rows and header metadata count terminal leaf subtests once; parent test events are not double-counted. Package build failures, process stderr, malformed JSON, and unknown Go event actions remain global diagnostics and are included in the Markdown summary. An invalid or unknown event forces a failed result even if the child process exits with code `0`. Relative source paths and clickable links are resolved from the configured `cwd` workspace root.

## Project configuration and overrides

`runGoTests()` automatically reads the root `loglensreporter.config.json`:

```json
{
  "borderStyle": "double",
  "go": {
    "directory": "backend",
    "title": "NATIVE GO TEST RUN",
    "borderStyle": "single"
  }
}
```

Go uses `single`; other providers keep the root `double` value. Options passed to `runGoTests()` override both levels.

## Exit handling

Always propagate the returned exit code:

```ts
const result = await runGoTests();
process.exitCode = result.exitCode;
```

The result includes:

```ts
interface GoRunResult {
  status: TestStatus;
  exitCode: number;
  signal: NodeJS.Signals | null;
  totals: TestTotals;
}
```

`SIGINT`, `SIGTERM`, and `SIGHUP` are forwarded to the Go child process. Conventional signal exit codes are returned when Go exits because of a signal.

See the [configuration reference](configuration.md) for shared options.
