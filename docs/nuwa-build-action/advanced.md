# Advanced Configuration

Control which Python versions to build, architectures to target, and other build options using standard `cibuildwheel` environment variables.

## Python Versions

Use `CIBW_BUILD` to specify which Python versions to build:

```yaml
steps:
  - name: Build wheels
    uses: martineastwood/nuwa-build-action@v1
    with:
      nim-version: "2.0.0"
    env:
      # Build specific Python versions
      CIBW_BUILD: "cp310-* cp311-* cp312-* cp313-*"
```

### Python Version Identifiers

| Identifier | Python Version |
| ---------- | -------------- |
| `cp39-*`   | Python 3.9     |
| `cp310-*`  | Python 3.10    |
| `cp311-*`  | Python 3.11    |
| `cp312-*`  | Python 3.12    |
| `cp313-*`  | Python 3.13    |
| `cp314-*`  | Python 3.14    |
| `cp314t-*` | Python 3.14 (free-threaded) |

## Skip Specific Builds

Use `CIBW_SKIP` to exclude certain builds:

```yaml
env:
  # Skip PyPy and musllinux builds
  CIBW_SKIP: "pp* *-musllinux_*"

  # Skip Windows 32-bit
  CIBW_SKIP: "win32 *-win32"

  # Skip specific Python versions
  CIBW_SKIP: "cp39-*"
```

## Architecture Targets

### macOS Universal Wheels

Build universal wheels (x86_64 + arm64) for macOS:

```yaml
env:
  CIBW_ARCHS_MACOS: "x86_64 arm64"
```

### Linux Architectures

```yaml
env:
  # Build specific architectures
  CIBW_ARCHS_LINUX: "x86_64 aarch64"
```

### Windows Architectures

```yaml
env:
  # Build 64-bit only (default)
  CIBW_ARCHS_WINDOWS: "AMD64"
```

## Build Options

### Before Build

Run commands before building (useful for installing Nimble dependencies):

```yaml
env:
  # Linux
  CIBW_BEFORE_ALL_LINUX: "yum install -y libffi-devel && nimble install deps"

  # macOS
  CIBW_BEFORE_ALL_MACOS: "brew install libffi && nimble install deps"

  # Windows
  CIBW_BEFORE_ALL_WINDOWS: "nimble install deps"
```

!!! warning
    On Linux, the action sets `CIBW_BEFORE_ALL_LINUX` to install Nim. If you override this, Nim won't be installed and the build will fail. Use `CIBW_BEFORE_BUILD` instead for Linux.

### Test Commands

Test wheels after building:

```yaml
env:
  CIBW_TEST_COMMAND: "pytest {project}/tests"
  CIBW_TEST_REQUIRES: "pytest"
```

### Build Verification

```yaml
env:
  # Skip building if the wheel already exists
  CIBW_BUILD_VERBOSITY: "1"
```

## Complete Example

```yaml
name: Build and Publish

on:
  push:
    tags:
      - "v*"

jobs:
  build_wheels:
    name: Build wheels on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]

    steps:
      - uses: actions/checkout@v4

      - name: Build wheels
        uses: martineastwood/nuwa-build-action@v1
        with:
          nim-version: "2.0.0"
          cibw-version: "2.22.0"
        env:
          # Python 3.10-3.13 only
          CIBW_BUILD: "cp310-* cp311-* cp312-* cp313-*"
          # Skip PyPy and musllinux
          CIBW_SKIP: "pp* *-musllinux_*"
          # Universal macOS wheels
          CIBW_ARCHS_MACOS: "x86_64 arm64"
          # Test after building
          CIBW_TEST_COMMAND: "pytest {project}/tests"
          CIBW_TEST_REQUIRES: "pytest"

      - uses: actions/upload-artifact@v4
        with:
          name: cibw-wheels-${{ matrix.os }}-${{ strategy.job-index }}
          path: ./wheelhouse/*.whl
```

## Environment Variable Reference

See the [cibuildwheel documentation](https://cibuildwheel.pypa.io/en/stable/settings/) for all available options.

Common variables:

- `CIBW_BUILD` - Build only these Python/OS combinations
- `CIBW_SKIP` - Skip these Python/OS combinations
- `CIBW_ARCHS_LINUX` - Linux architectures (x86_64, aarch64)
- `CIBW_ARCHS_MACOS` - macOS architectures (x86_64, arm64, universal2)
- `CIBW_ARCHS_WINDOWS` - Windows architectures (AMD64, x86, ARM64)
- `CIBW_BEFORE_ALL` - Commands before building all wheels
- `CIBW_BEFORE_BUILD` - Commands before building each wheel
- `CIBW_TEST_COMMAND` - Command to test wheels
- `CIBW_TEST_REQUIRES` - Python packages needed for testing
