# public-actions-library

Reusable GitHub Actions for CI/CD pipelines

## Features
- Build, test, and release automation for Maven projects
- Semantic versioning and release tagging
- Snapshot publishing to GitHub Packages

## Usage
Reference these actions in your workflow:

```yaml
jobs:
  build:
    uses: team-appark/public-actions-library/.github/workflows/build-library-maven.yaml@main
    with:
      java-version: '25'
      # ...other inputs...
```

## Maintainers
This library is maintained by team-appark.
