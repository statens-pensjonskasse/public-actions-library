# Build & Release Node/TypeScript Action

Reusable workflow for building, testing, and releasing GitHub Actions with automated semantic versioning.

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
    uses: statens-pensjonskasse/public-actions-library/.github/workflows/build-action-node.yaml@SHA # v1.0.0
    permissions:
      contents: write
    with:
      node-version: '24'
```

## What It Does

1. **Validates** - Checks inputs and verifies scripts exist in `package.json`
2. **Builds** - Installs dependencies, lints, builds, and tests your action
3. **Verifies** - Ensures compiled files are committed and in sync
4. **Releases** - Creates GitHub release with semantic versioning (main branch only)
5. **Tags** - Updates `v1` and `v1.2` tags for easy consumption

## Requirements

- `package.json` in repository root
- Scripts defined in `package.json`:
  - **Required**: `build` - needed to validate committed built files are in sync
  - **Optional**: `test`, `lint` - automatically skipped if not present
- Compiled JavaScript committed (e.g., `dist/index.js`)

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `node-version` | ✅ | - | Node.js version |
| `package-manager` | | `'npm'` | `npm`, `yarn`, or `pnpm` |
| `build-script` | | `'build'` | Script name for building |
| `test-script` | | `'test --if-present'` | Script name for testing (skipped if not in package.json) |
| `lint-script` | | `'lint --if-present'` | Script name for linting (skipped if not in package.json) |
| `require-release-flag` | | `false` | Only release if commit contains `[release]` |
| `minor-pattern` | | `'/^(feat\|feature)/'` | Regex for minor version bump |
| `major-pattern` | | `'/(\!: \|BREAKING CHANGE)/'` | Regex for major version bump |
| `notes-start-tag` | | - | Tag to start release notes from |

## Outputs

| Output | Description |
|--------|-------------|
| `published` | `'true'` if release was created, `'false'` otherwise |
| `new-version` | Version released (e.g., `v1.2.3`) or empty string |

## Examples

### Using pnpm
```yaml
jobs:
  build-and-release:
    uses: statens-pensjonskasse/public-actions-library/.github/workflows/build-action-node.yaml@SHA # v1.0.0
    permissions:
      contents: write
    with:
      node-version: '24'
      package-manager: 'pnpm'
```

### Custom script names
```yaml
with:
  build-script: 'compile'
  test-script: 'test:ci'
  lint-script: 'check'
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

### Version Tags

Creates three tags:
- `v1.2.3` - Exact version (immutable)
- `v1.2` - Latest patch (updated on each patch)
- `v1` - Latest minor+patch (updated on each release)

Users can reference:
```yaml
uses: your-org/action@v1        # Always latest (recommended)
uses: your-org/action@v1.2      # Pin to minor version
uses: your-org/action@v1.2.3    # Pin to exact version
```

## Built Files Validation

The workflow ensures compiled files are up-to-date by checking for uncommitted changes after build.

**If validation fails:**
```bash
npm run build
git add dist/
git commit -m "chore: rebuild action"
git push
```

This is required because GitHub Actions run the committed JavaScript, not TypeScript source.
