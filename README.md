# Clikd Release Action

Automated semantic releases with AI-powered changelogs for monorepos and multi-language projects.

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Clikd%20Release-purple?logo=github)](https://github.com/marketplace/actions/clikd-release-action)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Features

- **Conventional Commits Analysis** - Automatically determines version bumps (major/minor/patch) from commit messages
- **Multi-Language Support** - Rust, Node.js, Python, Go, Elixir, C#
- **Monorepo Support** - Handles multiple packages with dependency graph resolution
- **AI-Powered Changelogs** - Optional AI enhancement using Claude API
- **Automatic GitHub Releases** - Creates releases with generated changelogs
- **Zero Configuration** - Works out of the box with sensible defaults

## Quick Start

```yaml
name: Release

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: clikd-inc/release-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Prerequisites

Before using this action, initialize clikd in your repository:

```bash
# Install clikd
curl -fsSL https://clikd.dev/install.sh | sh

# Initialize release management
clikd release init
```

This creates a `.clikd/release.toml` configuration file.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | Version of clikd to install | No | `latest` |
| `push` | Push commits and tags to remote | No | `true` |
| `github-release` | Create GitHub releases | No | `true` |
| `anthropic-api-key` | API key for AI changelogs | No | - |
| `github-token` | GitHub token for releases | No | `github.token` |
| `working-directory` | Working directory | No | `.` |
| `dry-run` | Run without pushing | No | `false` |
| `extra-args` | Additional clikd arguments | No | - |

## Outputs

| Output | Description |
|--------|-------------|
| `released` | `true` if releases were created |
| `releases` | JSON array of released packages |
| `release-count` | Number of packages released |

### Releases Output Format

```json
[
  {
    "name": "my-package",
    "old_version": "1.0.0",
    "new_version": "1.1.0",
    "tag": "my-package-v1.1.0"
  }
]
```

## Examples

### Basic Usage

```yaml
- uses: clikd-inc/release-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### With AI-Powered Changelogs

```yaml
- uses: clikd-inc/release-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Dry Run (Testing)

```yaml
- uses: clikd-inc/release-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    dry-run: true
```

### Monorepo with Conditional Jobs

```yaml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    outputs:
      released: ${{ steps.release.outputs.released }}
      releases: ${{ steps.release.outputs.releases }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - id: release
        uses: clikd-inc/release-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}

  publish:
    needs: release
    if: needs.release.outputs.released == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Publish released packages
        run: |
          echo "Released packages:"
          echo '${{ needs.release.outputs.releases }}' | jq -r '.[].name'
```

### Custom Working Directory

```yaml
- uses: clikd-inc/release-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    working-directory: packages/core
```

### Skip GitHub Releases

```yaml
- uses: clikd-inc/release-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    github-release: false
```

## How It Works

1. **Checkout** - The action requires a full git history (`fetch-depth: 0`)
2. **Install** - Downloads and installs the clikd CLI
3. **Analyze** - Parses conventional commits since last release
4. **Bump** - Determines appropriate version bumps per package
5. **Changelog** - Generates changelog entries (optionally enhanced with AI)
6. **Commit** - Creates a release commit with version bumps
7. **Tag** - Creates git tags for each released package
8. **Push** - Pushes commits and tags to remote
9. **Release** - Creates GitHub releases with changelogs

## Conventional Commits

Version bumps are determined by commit prefixes:

| Prefix | Version Bump | Example |
|--------|--------------|---------|
| `feat:` | Minor | `feat: add new API endpoint` |
| `feat!:` or `BREAKING CHANGE:` | Major | `feat!: redesign API` |
| `fix:` | Patch | `fix: resolve memory leak` |
| `perf:` | Patch | `perf: optimize query` |
| `refactor:` | Patch | `refactor: simplify logic` |
| `chore:`, `docs:`, `ci:`, `test:` | No bump | `docs: update README` |

## Supported Languages

| Language | Version Files |
|----------|---------------|
| Rust | `Cargo.toml` |
| Node.js | `package.json` |
| Python | `pyproject.toml`, `setup.py` |
| Go | `go.mod` |
| Elixir | `mix.exs` |
| C# | `*.csproj` |

## Troubleshooting

### No releases created

- Ensure commits follow [Conventional Commits](https://www.conventionalcommits.org/) format
- Check that `.clikd/release.toml` exists (run `clikd release init`)
- Verify `fetch-depth: 0` in checkout action

### Permission denied

- Add `permissions: contents: write` to your workflow
- For fine-grained PATs, ensure "Contents: Read and write" permission

### AI changelog not working

- Verify `ANTHROPIC_API_KEY` is set correctly
- Check API key has sufficient credits/quota

## License

MIT License - see [LICENSE](LICENSE) for details.

## Contributing

Contributions welcome! Please read our [Contributing Guide](CONTRIBUTING.md) first.

## Links

- [clikd CLI Documentation](https://github.com/clikd-inc/cli)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Semantic Versioning](https://semver.org/)
