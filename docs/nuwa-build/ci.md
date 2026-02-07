# Continuous Integration

The `nuwa new` template includes a pre-configured GitHub Actions workflow for automated cross-platform wheel building and PyPI publishing.

## What's Included

The workflow uses a custom composite action that integrates with [cibuildwheel](https://github.com/pypa/cibuildwheel) to build wheels across:

- **Platforms**: Linux (manylinux), macOS, Windows
- **Python versions**: 3.9, 3.10, 3.11, 3.12, 3.13, 3.14
- **Architectures**: x86_64, arm64 (Apple Silicon)

## How It Works

The custom action handles platform-specific Nim compiler installation:

| Platform  | Installation Method                          |
| --------- | -------------------------------------------- |
| **Linux** | Installs Nim in Docker container via tar.xz  |
| **Windows** | Uses Chocolatey (`choco install nim`)      |
| **macOS** | Uses choosenim installer                     |

## First-Time Setup

### 1. Configure Trusted Publishing on PyPI

Go to https://pypi.org/manage/account/publishing/ and add a new publisher with:

- **PyPI Project Name**: Your package name
- **Owner**: Your GitHub username/organization
- **Repository name**: Your repository name
- **Workflow name**: `publish.yml`

### 2. Push a Version Tag

```bash
git tag v1.0.0
git push origin v1.0.0
```

## Manual Workflow Trigger

You can manually trigger the workflow from the GitHub Actions tab in your repository, useful for testing CI before release.

## Customization

Edit `.github/workflows/publish.yml` to customize:

```yaml
- name: Build wheels
  uses: martineastwood/nuwa-build-action@v1
  with:
    nim-version: "2.2.0"      # Nim version to install
    cibw-version: "2.22.0"     # cibuildwheel version
```
