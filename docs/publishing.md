# Publishing releases

Publish a beta first. Validate it in an unrelated repository before assigning the stable `latest` tag.

## npm account preparation

```bash
npm login
npm whoami
```

Direct publication requires the npm account's current publishing authentication requirements, including 2FA when configured.

## Release checklist

```bash
pnpm run check
pnpm test
pnpm run build
npm pack --dry-run
npm publish --dry-run
```

Review the tarball list for secrets, local paths, fixtures, generated caches, and missing declarations.
The build minifies every generated JavaScript file without publishing source maps and fails if CommonJS or ESM export shapes change. Consumer validation must still load every public entry point from the packed artifact.

## Publish the beta

The package version must be a prerelease such as `0.1.0-beta.0`.

```bash
npm publish --tag beta
```

Do not use plain `npm publish` for a prerelease because npm assigns the `latest` tag by default.

Verify the registry state:

```bash
npm view log-lens-reporter version dist-tags
```

Consumers install the beta with:

```bash
pnpm add -D log-lens-reporter@beta
```

## Publish another beta

Published versions are immutable. Increment the prerelease without creating a Git commit or tag automatically:

```bash
npm version prerelease --preid=beta --no-git-tag-version
```

Then repeat the release checklist and publish with `--tag beta`.

## Publish stable

After external beta validation:

```bash
npm version 0.1.0 --no-git-tag-version
pnpm run check
pnpm test
pnpm run build
npm publish --dry-run
npm publish
```

Plain `npm publish` assigns the stable version to `latest`.

## Visibility

`log-lens-reporter` is an unscoped package name. Unscoped npm packages are public. Publishing the package exposes the tarball contents even if the source repository is private.

For automated releases, prefer npm trusted publishing from a supported CI provider instead of storing a long-lived npm token.
