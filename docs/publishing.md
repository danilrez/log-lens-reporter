# Publishing releases

Validate every release in an unrelated repository before publishing it with the stable `latest` tag. The private repository is the release source of truth. Sync public documentation only from the committed release state on private `main`.

## Repository release sequence

1. Verify that private `main` is clean, up to date, and green in CI.
2. Confirm the exact stable version in `package.json`. A release-line merge may have promoted an existing `alpha` or `beta` version automatically; do not apply a second bump. For a manually prepared release, set the requested version, update the changelog and documentation, run the full release checklist, and commit the private release state.
3. Sync public documentation from the committed private `main` state. Record the same version in the public changelog. Review the full public diff and commit it.
4. Wait for explicit user confirmation after both release commits have been reviewed.
5. Only then create tags, create a GitHub release, or publish to npm.

Do not sync the public repository from a feature or release branch.

## Automated branch versioning

Merged pull requests update `package.json` through `.github/workflows/bump-version.yml`. The shared calculation lives in `.github/actions/version-bump/calculate-version.mts` and has regression tests in `tests/actions/version-bump.spec.mts`.

| Source branch                   | Version intent  |
| ------------------------------- | --------------- |
| `feature/*`                     | minor (`X.Y.0`) |
| `fix/*`, `bugfix/*`, `hotfix/*` | patch (`X.Y.Z`) |

The target branch selects the release channel:

- `main` receives a stable version. An existing `alpha` or `beta` line is promoted without another base-version bump.
- `feature/**` receives `-alpha.01`, then increments the prerelease number for later merges.
- `release/vMAJOR.MINOR.PATCH` defines the beta version line. Its first merge receives `-beta.01`, then later merges on that same line increment the prerelease number.

Source branch intent selects stable bumps on `main` and the initial alpha base on `feature/**`. The explicit version in a `release/vMAJOR.MINOR.PATCH` target branch controls its beta line.

Only merged pull requests from recognized branches in the same repository are eligible. Other branches can merge without changing the package version. Version-bump commits are serialized per target branch and are pushed by the GitHub Actions bot.

Examples starting from `1.0.1`:

| Merge                                      | Result           |
| ------------------------------------------ | ---------------- |
| `feature/reporter` → `feature/reporter-v2` | `1.1.0-alpha.01` |
| `fix/output` → the same feature branch     | `1.1.0-alpha.02` |
| `feature/reporter-v2` → `release/v1.1.0`   | `1.1.0-beta.01`  |
| `fix/output` → the same release branch     | `1.1.0-beta.02`  |
| `release/v1.1.0` → `main`                  | `1.1.0`          |

Direct merges into `main` use the source intent directly: a `feature/*` merge produces the next minor stable version and a `fix/*`, `bugfix/*`, or `hotfix/*` merge produces the next patch stable version. A merge from an ordinary branch does not trigger the bump workflow.

## npm account preparation

```bash
npm login
npm whoami
```

Direct publication requires the npm account's current publishing authentication requirements, including 2FA when configured.

Do not continue when `npm whoami` fails.

## Release checklist

Use Node.js 24, as pinned in `.nvmrc`, for repository checks and builds. The published package itself continues to support Node.js 20 and newer.

```bash
pnpm run check
pnpm test
pnpm run build
pnpm pack --pack-destination <temporary-directory>
npm publish --dry-run
```

Review the tarball file list for secrets, local paths, fixtures, generated caches, and missing type declarations.

The build minifies every generated JavaScript file without publishing source maps. It fails if CommonJS or ESM export shapes change. Consumer validation must also load every public entry point from the packed artifact.

## Publish stable

Confirm that `package.json` already contains the intended stable version. For the `1.0.0` release, set it only if it has not already been prepared:

```bash
npm version 1.0.0 --no-git-tag-version
```

Then run the full checklist against that exact version:

```bash
pnpm run check
pnpm test
pnpm run build
npm pack --pack-destination <temporary-directory>
npm publish --dry-run
```

Install and verify the final packed artifact. Sync and review both release commits, then wait for explicit approval. After approval, run `npm publish`. This command assigns the stable version to `latest`.

Verify the registry state after publishing:

```bash
npm view log-lens-reporter version dist-tags
```

Published versions are immutable. Increment the package version before preparing another release.

## Visibility

`log-lens-reporter` is an unscoped package name. Unscoped npm packages are public. Publishing the package exposes the tarball contents even if the source repository is private.

For automated releases, prefer npm trusted publishing from a supported CI provider instead of storing a long-lived npm token.
