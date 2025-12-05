# Clikd Release Action

Automated semantic releases with AI-powered changelogs for monorepos and multi-language projects.

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Clikd%20Release-purple?logo=github)](https://github.com/marketplace/actions/clikd-release-action)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Features

- **Conventional Commits Analysis** - Automatically determines version bumps (major/minor/patch) from commit messages
- **Multi-Language Support** - Rust, Node.js, Python, Go, Elixir, C#
- **Monorepo Support** - Handles multiple packages with dependency graph resolution
- **AI-Powered Changelogs** - Optional AI enhancement using Claude API
- **PR-Based Workflow** - Creates release PRs for review before publishing
- **Zero Configuration** - Works out of the box with sensible defaults

## How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│  1. Action runs clikd release prepare --ci                     │
│     ├── Analyzes conventional commits                          │
│     ├── Bumps versions (major/minor/patch)                     │
│     ├── Generates changelogs (with optional AI)                │
│     ├── Creates release manifest in clikd/releases/            │
│     └── Creates Pull Request                                   │
├─────────────────────────────────────────────────────────────────┤
│  2. You review and merge the PR                                │
├─────────────────────────────────────────────────────────────────┤
│  3. GitHub App (clikd-bot) handles post-merge:                 │
│     ├── Creates Git tags                                       │
│     ├── Creates GitHub Releases                                │
│     └── Cleans up manifest files                               │
└─────────────────────────────────────────────────────────────────┘
```

## Quick Start

```yaml
name: Release

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

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

This creates a `clikd/` directory with configuration files.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | Version of clikd to install | No | `latest` |
| `bump` | Version bump type: `major`, `minor`, `patch`, `auto` | No | `auto` |
| `projects` | Per-project bumps (e.g., `gate:major,rig:minor`) | No | - |
| `anthropic-api-key` | API key for AI changelogs | No | - |
| `github-token` | GitHub token for PR creation | No | `github.token` |
| `working-directory` | Working directory | No | `.` |
| `dry-run` | Run without creating PR | No | `false` |
| `git-user-name` | Git user name for commits | No | `github-actions[bot]` |
| `git-user-email` | Git user email for commits | No | `github-actions[bot]@users.noreply.github.com` |

## Outputs

| Output | Description |
|--------|-------------|
| `pr-created` | `true` if a release PR was created |
| `pr-url` | URL of the created pull request |
| `pr-number` | Number of the created pull request |
| `releases` | JSON array of packages to be released |
| `release-count` | Number of packages in the release |

### Releases Output Format

```json
[
  {
    "name": "my-package",
    "old_version": "1.0.0",
    "new_version": "1.1.0"
  }
]
```

## Examples

### Basic Usage (Auto-detect bump from commits)

```yaml
- uses: clikd-inc/release-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Force Major Release

```yaml
- uses: clikd-inc/release-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    bump: major
```

### Per-Project Bumps (Monorepo)

```yaml
- uses: clikd-inc/release-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    projects: 'api:major,web:minor,shared:patch'
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

### Using Outputs

```yaml
- id: release
  uses: clikd-inc/release-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}

- name: Show PR
  if: steps.release.outputs.pr-created == 'true'
  run: |
    echo "Release PR: ${{ steps.release.outputs.pr-url }}"
    echo "Packages: ${{ steps.release.outputs.release-count }}"
```

### Auto-Merge Release PRs

```yaml
- id: release
  uses: clikd-inc/release-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}

- name: Enable auto-merge
  if: steps.release.outputs.pr-created == 'true'
  run: gh pr merge --auto --squash "${{ steps.release.outputs.pr-number }}"
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Custom Git Identity

```yaml
- uses: clikd-inc/release-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    git-user-name: 'Release Bot'
    git-user-email: 'release-bot@example.com'
```

### Specific clikd Version

```yaml
- uses: clikd-inc/release-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    version: '0.6.0'
```

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

### No PR created

- Ensure commits follow [Conventional Commits](https://www.conventionalcommits.org/) format
- Check that `clikd/` directory exists (run `clikd release init`)
- Verify `fetch-depth: 0` in checkout action

### Permission denied

- Add `permissions: contents: write` and `pull-requests: write` to your workflow
- For fine-grained PATs, ensure "Contents: Read and write" and "Pull requests: Read and write" permissions

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
