# public-actions-library

Reusable GitHub Actions workflows for CI/CD pipelines with automated building, testing, and semantic versioning.

## Available Workflows

### Maven Library (`build-library-maven.yaml`)
Build, test, and release Maven libraries with automatic version management and GitHub Packages publishing.

```yaml
jobs:
  build:
    uses: statens-pensjonskasse/public-actions-library/.github/workflows/build-library-maven.yaml@main
    with:
      java-version: '21'
    secrets: inherit
```

**Use for:** Java/Kotlin Maven projects  
**Outputs:** JAR/WAR artifacts published to GitHub Packages  
**Versioning:** Updates pom.xml with new versions  
📖 [Documentation](docs/build-library-maven.md)

---

### Node/TypeScript Action (`build-action-node.yaml`)
Build, test, and release TypeScript GitHub Actions with built-file validation and version tag management.

```yaml
jobs:
  build:
    uses: statens-pensjonskasse/public-actions-library/.github/workflows/build-action-node.yaml@main
    with:
      node-version: '20'
      package-manager: 'npm'  # or 'yarn', 'pnpm'
    secrets: inherit
```

**Use for:** TypeScript/JavaScript GitHub Actions  
**Outputs:** GitHub releases with moving version tags (v1, v1.2)  
**Validation:** Ensures compiled JavaScript is committed  
📖 [Documentation](docs/build-action-node.md)

---

## Common Features

### Semantic Versioning
All workflows use conventional commits to automatically calculate versions:

| Commit Type | Example | Version Bump |
|-------------|---------|--------------|
| **Patch** | `fix: correct bug`<br>`docs: update README`<br>`chore: update deps` | 1.0.0 → 1.0.1 |
| **Minor** | `feat: add new feature`<br>`feature: implement X` | 1.0.0 → 1.1.0 |
| **Major** | `feat!: breaking change`<br>`feat: BREAKING CHANGE description` | 1.0.0 → 2.0.0 |

### Release Control
Control when releases are created:

**Skip Release:**
```bash
git commit -m "docs: update README [skip release]"
```
Markers: `[skip release]`, `[release skip]`, `[skip publish]`, `[publish skip]`

**Require [release] Flag:**
```yaml
with:
  require-release-flag: true
```
Then only commits with `[release]` will trigger a release.

**Branch Protection:**
- Releases only created from `main` branch
- Bot commits automatically skipped

### Custom Version Patterns
Customize what triggers version bumps:

```yaml
with:
  minor-pattern: '/^(feat|feature|enhancement)/'
  major-pattern: '/^(breaking|major)/'
```

### Release Notes
Automatically generated from commit history:

```yaml
with:
  notes-start-tag: 'v1.0.0'  # Start notes from this tag
```

## Permissions Required

All workflows need `contents: write` to create releases:

```yaml
jobs:
  build:
    uses: statens-pensjonskasse/public-actions-library/.github/workflows/[workflow].yaml@main
    permissions:
      contents: write
    secrets: inherit
```

## Quick Start

1. Choose the appropriate workflow for your project type
2. Add the workflow reference to your `.github/workflows/ci.yaml`
3. Ensure you're using conventional commit messages
4. Push to `main` branch to trigger a release

That's it! No secrets to configure, no complex setup required.

## Maintainers
This library is maintained by team-appark.
