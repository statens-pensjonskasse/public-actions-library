# public-actions-library

Reusable GitHub Actions workflows for building, testing, and releasing with automated semantic versioning.

## Workflows

### Node/TypeScript Action
Build and release GitHub Actions with version tag management.

```yaml
jobs:
  build-and-release:
    uses: statens-pensjonskasse/public-actions-library/.github/workflows/build-action-node.yaml@SHA # v1.0.0
    permissions:
      contents: write
    with:
      node-version: '24'
```

📖 [Documentation](docs/build-action-node.md)

### Maven Library
Build and release Maven libraries with GitHub Packages publishing.

```yaml
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

📖 [Documentation](docs/build-library-maven.md)

## Semantic Versioning

Version bumps determined by commit messages:

- `feat:` → Minor (1.2.0 → 1.3.0)
- `fix:` → Patch (1.2.0 → 1.2.1)
- `feat!:` or `BREAKING CHANGE` → Major (1.2.0 → 2.0.0)

## Release Control

Releases are automatically created from `main` branch on every push, unless:

**Skip markers** - Add to commit message to prevent release:
- `[skip release]`
- `[release skip]`
- `[skip publish]`
- `[publish skip]`

```bash
git commit -m "docs: update README [skip release]"
```

**Bot commits** - Automatically skipped

**Require [release] flag** - Only create releases when explicitly requested:
```yaml
with:
  require-release-flag: true
```

Then only commits with `[release]` in the message will trigger a release:
```bash
git commit -m "feat: new feature [release]"
```

