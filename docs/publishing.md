# Publishing releases

Validate every release in an unrelated repository before publishing it with the stable `latest` tag. The private repository is the release source of truth. Sync public documentation only from the committed release state on private `main`.

## Repository release sequence

1. Verify that private `main` is clean, up to date, and green in CI.
2. Set the exact release version. Update the changelog and documentation, run the full release checklist, and commit the private release state.
3. Sync public documentation from the committed private `main` state. Record the same version in the public changelog. Review the full public diff and commit it.
4. Wait for explicit user confirmation after both release commits have been reviewed.
5. Only then create tags, create a GitHub release, or publish to npm.

Do not sync the public repository from a feature or release branch.

## npm account preparation

```bash
npm login
npm whoami
```

Direct publication requires the npm account's current publishing authentication requirements, including 2FA when configured.

Do not continue when `npm whoami` fails.

## Release checklist

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
