# Node/TypeScript GitHub Action Workflow

Build, test, and release TypeScript GitHub Actions with automated version tag management.

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
    uses: statens-pensjonskasse/public-actions-library/.github/workflows/build-action-node.yaml@main
    permissions:
      contents: write
    secrets: inherit
```

## Prerequisites

- `package.json` with `build` and `test` scripts (and `lint` if configured)
- All configured scripts must be defined in package.json's `scripts` section
- Compiled JavaScript files committed to repository (typically in `dist/` or `lib/`)

## Key Features

✅ **Built File Validation** - Ensures compiled JS is in sync with TypeScript  
✅ **Version Tag Management** - Auto-updates v1, v1.2 tags for easy consumption  
✅ **Package Manager Support** - npm, yarn, or pnpm  
✅ **Security Hardened** - Input validation prevents command injection

## Configuration

### Inputs

| Input | Default                       | Description |
|-------|-------------------------------|-------------|
| `node-version` | `'24'`                        | Node.js version |
| `package-manager` | `'npm'`                       | Package manager: `npm`, `yarn`, or `pnpm` |
| `build-script` | `'build'`                     | npm script for building |
| `test-script` | `'test'`                      | npm script for testing |
| `lint-script` | `'lint'`                      | npm script for linting (optional) |
| `require-release-flag` | `false`                       | Require `[release]` in commit message |
| `minor-pattern` | `'/^(feat\|feature)/'`        | Pattern for minor version bump |
| `major-pattern` | `'/(\!: \|BREAKING CHANGE)/'` | Pattern for major version bump |
| `notes-start-tag` | -                             | Starting tag for release notes |

### Outputs

| Output | Description |
|--------|-------------|
| `published` | `true` if a release was created |
| `new-version` | The version that was released (e.g., `1.2.3`) |

## Examples

### With pnpm
```yaml
with:
  package-manager: 'pnpm'
  node-version: '24'
```

### Custom Scripts
```yaml
with:
  build-script: 'build:production'
  test-script: 'test:ci'
  lint-script: 'lint:all'
```

### Require [release] Flag
```yaml
with:
  require-release-flag: true
```

## Workflow Steps

1. **Validate** - Security check on inputs and verify all required scripts exist in package.json
2. **Prepare** - Determine if should release
3. **Build** - Install deps → Lint → Build → Test → Validate compiled files
4. **Release** - Create release → Update version tags (v1, v1.2)
5. **Summary** - Display workflow results

## Built Files Validation

**Critical:** GitHub Actions require compiled JavaScript to be committed.

After building, the workflow checks:
```bash
git status --porcelain  # Must be empty
```

If validation fails:
```
Built files are not in sync with source code!
Please rebuild locally and commit:
  npm run build
  git add dist/
  git commit -m 'chore: rebuild action'
```

## Version Tags

The workflow creates and maintains version tags:

- `v1.2.3` - Exact version (immutable)
- `v1.2` - Latest patch (moving tag)
- `v1` - Latest minor (moving tag)

Users reference your action with:
```yaml
- uses: your-org/your-action@v1      # Recommended
- uses: your-org/your-action@v1.2    # Pin to minor
- uses: your-org/your-action@v1.2.3  # Pin to exact
```

## Repository Structure

```
your-action/
├── .github/workflows/ci.yaml
├── src/
│   ├── index.ts
│   └── main.ts
├── dist/              # ← Must be committed!
│   └── index.js
├── action.yml
├── package.json
└── tsconfig.json
```

## Troubleshooting

### Script not found in package.json
If validation fails with "Required script 'X' not found in package.json":
1. Check your package.json has the script defined:
   ```json
   "scripts": {
     "build": "tsc",
     "test": "jest",
     "lint": "eslint src/**/*.ts"
   }
   ```
2. Or adjust the workflow input to match your script name:
   ```yaml
   with:
     lint-script: 'lint:all'  # Match your actual script name
   ```
3. Or set the input to empty string to skip:
   ```yaml
   with:
     lint-script: ''  # Skip linting
   ```

### Built files not in sync
```bash
npm run build
git add dist/
git commit -m "chore: rebuild action"
```

### No release created
- Check you're on `main` branch
- Ensure no skip markers in commit (`[skip release]`)
- If `require-release-flag: true`, add `[release]` to commit

### Version not incrementing
Use conventional commits: `feat:`, `fix:`, `feat!:`  
See [README](../README.md#semantic-versioning) for details.

