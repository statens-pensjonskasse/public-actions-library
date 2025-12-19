# Build & Release Node/TypeScript Library

Reusable workflow for building, testing, and releasing npm libraries with automated semantic versioning and npm registry publishing.

## Quick Start

```yaml
# .github/workflows/build-and-release.yaml
name: Build and Release

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build-and-release:
    uses: statens-pensjonskasse/public-actions-library/.github/workflows/build-library-node.yaml@SHA # v1.0.0
    permissions:
      contents: write
      packages: write
    with:
      node-version: '24'
```

## What It Does

1. **Validates** - Checks inputs and verifies scripts exist in `package.json`
2. **Prepares** - Determines if a release should be created (runs in parallel with validate)
3. **Builds** - Installs dependencies, lints, builds, and tests your library
4. **Releases** - Updates `package.json` version, rebuilds, publishes to npm registry, creates GitHub release (main branch only)
5. **Summary** - Generates workflow execution summary

## Requirements

- `package.json` in repository root
- Optional scripts in `package.json`:
  - `build`, `test`, `lint` - executed via `npm run <script> --if-present`
- Repository secrets: `PUBLIC_ACTIONS_LIBRARY_APP_ID` and `PUBLIC_ACTIONS_LIBRARY_SSH_KEY` (for committing version changes)

**Note:** Publishing uses the built-in `npm publish` / `yarn publish` / `pnpm publish` command, not a package.json script.

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `node-version` | ✅ | - | Node.js version |
| `package-manager` | | `'npm'` | `npm`, `yarn`, or `pnpm` |
| `build-script` | | `'build --if-present'` | Script name for building (skipped if not in package.json) |
| `test-script` | | `'test --if-present'` | Script name for testing (skipped if not in package.json) |
| `lint-script` | | `'lint --if-present'` | Script name for linting (skipped if not in package.json) |
| `publish-script` | | `'publish'` | Command passed to package manager (e.g., `npm publish`, `yarn publish`) |
| `npm-registry` | | `'https://npm.pkg.github.com'` | npm registry URL for publishing |
| `require-release-flag` | | `false` | Only release if commit contains `[release]` |
| `minor-pattern` | | `'/^(feat\|feature)/'` | Regex for minor version bump |
| `major-pattern` | | `'/(\!: \|BREAKING CHANGE)/'` | Regex for major version bump |
| `notes-start-tag` | | - | Tag to start release notes from |

## Outputs

| Output | Description |
|--------|-------------|
| `published` | `'true'` if release was created, `'false'` otherwise |
| `new-version` | Version released (e.g., `1.2.3`) or empty string |

## Examples

### Using pnpm with GitHub Packages
```yaml
jobs:
  build-and-release:
    uses: statens-pensjonskasse/public-actions-library/.github/workflows/build-library-node.yaml@SHA # v1.0.0
    permissions:
      contents: write
      packages: write
    with:
      node-version: '24'
      package-manager: 'pnpm'
      npm-registry: 'https://npm.pkg.github.com'
```

### Using npmjs.org registry
```yaml
jobs:
  build-and-release:
    uses: statens-pensjonskasse/public-actions-library/.github/workflows/build-library-node.yaml@SHA # v1.0.0
    permissions:
      contents: write
      packages: write
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
    with:
      node-version: '24'
      npm-registry: 'https://registry.npmjs.org'
```

### Custom script names
```yaml
with:
  build-script: 'compile'
  test-script: 'test:ci'
  lint-script: 'check'
  publish-script: 'release:publish'
```

### Require [release] flag
```yaml
with:
  require-release-flag: true  # Only release commits with [release] in message
```

## Release Behavior

Releases automatically from `main` branch. 
Use `[skip release]` in commit message to skip, or set `require-release-flag: true` to only release with `[release]` flag. 
See [Release Control](../README.md#release-control) for details.

### Version Bumping

Uses commit messages to determine version:
- **Major** (1.x.x → 2.0.0): `feat!:`, `BREAKING CHANGE`, or matches `major-pattern`
- **Minor** (1.2.x → 1.3.0): `feat:`, `feature:`, or matches `minor-pattern`
- **Patch** (1.2.3 → 1.2.4): All other commits (`fix:`, `chore:`, etc.)

See [Semantic Versioning](../README.md#semantic-versioning) for more details.

### Version Management

The workflow automatically manages `package.json` versions:

**On builds** (PRs, feature branches):
- Builds and tests the library without publishing
- No version changes are made

**On release** (main branch):
1. Calculates new semantic version based on commit messages
2. Updates `package.json` to release version → `1.2.3`
3. Rebuilds the library with the new version
4. Commits: `chore(release): Release av versjon 1.2.3 [skip actions]`
5. Publishes to npm registry
6. Creates GitHub release with tag `1.2.3`

## Package.json Setup

### Example Scripts

Build, test, and lint scripts in `package.json` are optional:

```json
{
  "name": "@your-org/your-library",
  "version": "1.0.0",
  "scripts": {
    "build": "tsc",
    "test": "jest",
    "lint": "eslint src/"
  }
}
```

**Note:** These scripts are executed via `npm run <script>` and will be skipped if not present (due to `--if-present` flag).

The `publish` command is different - it uses the built-in package manager command:
- Workflow runs: `npm publish` (or `yarn publish`, `pnpm publish`)
- No script needed in `package.json`

To customize publishing, you can:

1. **Use a custom script via `run`:**
```yaml
with:
  publish-script: 'run my-publish'  # Runs: npm run my-publish
```

```json
{
  "scripts": {
    "my-publish": "npm publish --access public"
  }
}
```

2. **Pass additional flags directly:**
```yaml
with:
  publish-script: 'publish --access public'  # Runs: npm publish --access public
```

### Publishing to GitHub Packages

To publish to GitHub Packages, configure your `package.json`:

```json
{
  "name": "@your-org/your-library",
  "version": "1.0.0",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  }
}
```


The workflow automatically sets `NODE_AUTH_TOKEN` environment variable for authentication.

### Publishing to npmjs.org

For npmjs.org, you'll need to add an authentication token:

1. Generate an npm token at https://www.npmjs.com/settings/[username]/tokens
2. Add it as a repository secret named `NPM_TOKEN`
3. Pass it to the workflow (authentication is handled automatically via `NODE_AUTH_TOKEN`)

```json
{
  "name": "your-library",
  "version": "1.0.0",
  "publishConfig": {
    "access": "public"
  }
}
```

## Authentication

The workflow uses:
- **`NODE_AUTH_TOKEN`**: Set to `github.token` by default for GitHub Packages, or custom token for other registries
- **GitHub App**: Used for committing version changes (requires `PUBLIC_ACTIONS_LIBRARY_APP_ID` and `PUBLIC_ACTIONS_LIBRARY_SSH_KEY` secrets)

## Differences from build-action-node

Unlike the GitHub Action workflow which creates moving version tags (v1, v1.2), libraries use:
- Standard semantic versioning (1.2.3)
- No "v" prefix in version tags
- Direct npm registry publishing instead of committed distribution files
- Version updates committed back to the repository

### Build and Release Jobs

The workflow uses **separate build and release jobs** for:

**Build Job** (read-only permissions):
- Runs on every push (PRs and main branch)
- Validates the code builds and tests pass
- Uses minimal permissions (contents: read)
- Fast feedback on PRs

**Release Job** (write permissions, main branch only):
- Updates `package.json` version
- Rebuilds with the new version embedded
- Publishes to npm registry
- Commits version change and creates GitHub release
- Needs elevated permissions (contents: write, packages: write)

**Why rebuild in release job?**
- The version in `package.json` must be updated before building
- The new version gets embedded into built artifacts
- Similar to Maven's `mvn deploy` which includes compilation

**Trade-off:** This means building twice (once to test, once to release), but provides:
- ✅ Better security with permission isolation
- ✅ Clearer separation of concerns
- ✅ PR builds don't need write permissions
- ✅ Clear visibility of what succeeded/failed

This matches the Maven library workflow pattern.

## Troubleshooting

### Publish fails with authentication error

Ensure:
1. Workflow has `packages: write` permission
2. `npm-registry` input matches your `publishConfig.registry` in `package.json`
3. For external registries, authentication token is properly configured

### Version conflict

If the release fails because the version already exists:
- Check if a previous workflow run already published this version
- The workflow will skip publishing if the version already exists in the registry
- Ensure you're not manually publishing the same version

### Build succeeds but no release created

Check:
1. You're on the `main` branch
2. Commit message doesn't contain skip markers (`[skip release]`, etc.)
3. If `require-release-flag: true`, ensure commit contains `[release]`
4. Workflow was not triggered by a bot account

## Best Practices

1. **Use semantic commit messages**: `feat:`, `fix:`, `chore:` to get proper version bumps
2. **Test thoroughly**: Ensure tests pass before merging to main
3. **Use lockfiles**: Commit `package-lock.json`, `yarn.lock`, or `pnpm-lock.yaml`
4. **Keep build artifacts out of git**: Only commit source code, not built files
5. **Document breaking changes**: Use `feat!:` or `BREAKING CHANGE:` in commit messages
6. **Review releases**: Check the generated release notes after publishing

