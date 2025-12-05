# Maven Library Workflow

Build, test, and release Maven libraries with automatic version management and GitHub Packages publishing.

## Quick Start

```yaml
# .github/workflows/ci.yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build-and-release:
    uses: statens-pensjonskasse/public-actions-library/.github/workflows/build-library-maven.yaml@main
    permissions:
      contents: write
      packages: write
    secrets: inherit
```

## Prerequisites

- Maven project with `pom.xml`
- Standard Maven directory structure (`src/main/java`, etc.)

## Key Features

✅ **Automatic Versioning** - Updates pom.xml with semantic versions  
✅ **SNAPSHOT Publishing** - Publishes snapshots on non-release builds  
✅ **GitHub Packages** - Publishes artifacts to GitHub Packages  
✅ **Release Management** - Creates GitHub releases with changelogs

## Configuration

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `java-version` | `'21'` | Java version |
| `maven-version` | `'3.9.x'` | Maven version |
| `require-release-flag` | `false` | Require `[release]` in commit message |
| `minor-pattern` | `'/^(feat\|feature)/'` | Pattern for minor version bump |
| `major-pattern` | `'/(\!: \|BREAKING CHANGE)/'` | Pattern for major version bump |
| `notes-start-tag` | - | Starting tag for release notes |

### Outputs

| Output | Description |
|--------|-------------|
| `published` | `true` if a release was created |
| `new-version` | The version that was released (e.g., `1.2.3`) |

## Examples

### Custom Java Version
```yaml
with:
  java-version: '17'
```

### Require [release] Flag
```yaml
with:
  require-release-flag: true
```

## Workflow Steps

1. **Validate** - Check inputs and configuration
2. **Prepare** - Determine if should release
3. **Build** - Compile → Test → Package
4. **Publish** - Upload to GitHub Packages (SNAPSHOT or release)
5. **Release** - Create GitHub release (main branch only)

## Version Management

The workflow updates your `pom.xml` automatically:

- **On feature branches**: Publishes SNAPSHOT versions (e.g., `1.2.3-SNAPSHOT`)
- **On main branch**: Creates release versions (e.g., `1.2.3`)
- **After release**: Bumps to next SNAPSHOT version

## Repository Structure

```
your-library/
├── .github/workflows/ci.yaml
├── src/
│   ├── main/java/...
│   └── test/java/...
├── pom.xml
└── README.md
```

## Troubleshooting

### Build fails on test
Check your test configuration in `pom.xml` and ensure tests pass locally.

### Artifact not published
- Verify `packages: write` permission is granted
- Check GitHub Packages settings in your repository

### Version not incrementing
Use conventional commits: `feat:`, `fix:`, `feat!:`  
See [README](../README.md#semantic-versioning) for details.

