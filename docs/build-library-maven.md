# Build & Release Maven Library

Reusable workflow for building, testing, and releasing Maven libraries with automated semantic versioning and GitHub Packages publishing.

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
    uses: statens-pensjonskasse/public-actions-library/.github/workflows/build-library-maven.yaml@SHA # v1.0.0
    permissions:
      contents: write
      packages: write
    secrets: inherit
    with:
      java-version: '25'
```

## What It Does

1. **Prepares** - Determines if should create release
2. **Builds** - Compiles and tests with Maven
3. **Publishes Snapshot** - Uploads SNAPSHOT version to GitHub Packages (on PRs/branches)
4. **Releases** - Updates pom.xml, publishes release version, creates GitHub release (main branch only)
5. **Updates** - Bumps pom.xml to next SNAPSHOT version after release

## Requirements

- Maven project with `pom.xml` in repository root
- Maven wrapper (`mvnw`) in repository root
- Version in `pom.xml` must be `X.Y.Z-SNAPSHOT` format
- Repository secrets: `PUBLIC_ACTIONS_LIBRARY_APP_ID` and `PUBLIC_ACTIONS_LIBRARY_SSH_KEY`

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `java-version` | ✅ | - | Java SDK version |
| `java-distribution` | | `'temurin'` | Java distribution (temurin, zulu, etc.) |
| `build-goals` | | `'compile'` | Maven build goal(s) |
| `test-goals` | | `'verify'` | Maven test goal(s) |
| `publish-goals` | | `'deploy -Dmaven.test.skip -Dmdep.analyze.skip'` | Maven publish goal(s) |
| `require-release-flag` | | `false` | Only release if commit contains `[release]` |
| `minor-pattern` | | `'/^(feat\|feature)/'` | Regex for minor version bump |
| `major-pattern` | | `'/(\!: \|BREAKING CHANGE)/'` | Regex for major version bump |
| `notes-start-tag` | | - | Tag to start release notes from |

## Outputs

| Output | Description |
|--------|-------------|
| `published` | `'true'` if artifacts were published, `'false'` otherwise |
| `new-version` | Version released (e.g., `1.2.3`) or empty string |

## Examples

### Using Java 21
```yaml
jobs:
  build-and-release:
    uses: statens-pensjonskasse/public-actions-library/.github/workflows/build-library-maven.yaml@SHA # v1.0.0
    permissions:
      contents: write
      packages: write
    secrets: inherit
    with:
      java-version: '21'
```

### Custom Maven goals
```yaml
with:
  java-version: '25'
  build-goals: 'clean compile'
  test-goals: 'test'
  publish-goals: 'deploy'
```

### Require [release] flag
```yaml
with:
  java-version: '25'
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

The workflow automatically manages `pom.xml` versions:

**On SNAPSHOT builds** (PRs, feature branches):
- Keeps existing `X.Y.Z-SNAPSHOT` version
- Publishes to GitHub Packages with `-SNAPSHOT` suffix

**On release** (main branch):
1. Removes `-SNAPSHOT` from version → `1.2.3`
2. Commits: `chore(release): Release av versjon 1.2.3 [skip actions]`
3. Publishes release version to GitHub Packages
4. Creates GitHub release with tag `1.2.3`
5. Bumps to next SNAPSHOT → `1.2.4-SNAPSHOT`
6. Commits: `chore(snapshot): Bump snapshot-versjon til 1.2.4-SNAPSHOT`
