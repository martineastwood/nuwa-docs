# Continuous Integration

The `nuwa new` template includes a pre-configured GitHub Actions workflow for automated cross-platform wheel building and PyPI publishing.

## What's Included

The workflow uses a custom composite action that integrates with [cibuildwheel](https://github.com/pypa/cibuildwheel) to build wheels across:

- **Action support**: Linux manylinux x86_64/aarch64, macOS native Intel/Apple Silicon, and Windows x86_64
- **Generated default**: `ubuntu-latest`, `macos-latest`, and `windows-latest`; add native ARM64 or Intel runners when both architectures are required
- **Python versions**: 3.10, 3.11, 3.12, 3.13, 3.14 (regular CPython, not free-threaded)
- **Not included**: PyPy, musllinux, and `cp314t`

See the [support matrix](../support.md).

## How It Works

The custom action handles platform-specific Nim compiler installation:

| Platform  | Installation Method                          |
| --------- | -------------------------------------------- |
| **Linux** | Matching official `linux_x64` or `linux_arm64` Nim tarball inside the manylinux container (checksum-verified) |
| **Windows** | Chocolatey (`choco install nim`), including its MinGW dependency |
| **macOS** | Official `macosx_x64` or `macosx_arm64` archive matching the runner (checksum-verified) |

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

The generated workflow exposes `workflow_dispatch`, which runs its publication
job as well as its builds. Use it only when you intend to publish and have
configured the PyPI environment.

## Customization

Edit `.github/workflows/publish.yml` to customize:

```yaml
- name: Build wheels
  uses: martineastwood/nuwa-build-action@v1
  with:
    nim-version: "2.2.10"      # Nim version to install
    cibw-version: "4.2.0"     # cibuildwheel version
    package-dir: "."           # project directory
```
