# Go integration

The Go adapter runs `go test -json`, maps top-level tests and subtests to source files, preserves package build failures, forwards termination signals, and returns the child process exit evidence.

Failed test details are grouped directly below their `FAILED` file row. Package and build failures without a source test remain standalone diagnostics.

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

| Option            | Type                | Default                | Description                                           |
| ----------------- | ------------------- | ---------------------- | ----------------------------------------------------- |
| `cwd`             | `string`            | `process.cwd()`        | Workspace root used by the process and path mapping.  |
| `directory`       | `string`            | `'backend'`            | Go module directory passed to `go -C`.                |
| `packages`        | `string[]`          | `['./...']`            | Package patterns passed to `go test`.                 |
| `goBinary`        | `string`            | `'go'`                 | Go executable name or absolute path.                  |
| `goArguments`     | `string[]`          | `[]`                   | Extra flags inserted before package patterns.         |
| `env`             | `NodeJS.ProcessEnv` | inherited              | Environment overrides merged with `process.env`.      |
| `title`           | `string`            | `'NATIVE GO TEST RUN'` | Header title.                                         |
| `showDescription` | `boolean`           | `true`                 | Shows generated provider, base path, and test counts. |

The generated header metadata reflects the tests selected by `packages`, `-run`, and `-skip`. For example, a run limited to `./internal/store` reports that directory as its base path instead of the whole Go module.

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
