# Advanced Configuration

Control which Python versions to build, architectures to target, and other build options using standard `cibuildwheel` environment variables.

## Python Versions

Use `CIBW_BUILD` to specify which Python versions to build:

```yaml
steps:
  - name: Build wheels
    uses: martineastwood/nuwa-build-action@v1
    with:
      nim-version: "2.2.10"
    env:
      # Build specific Python versions
      CIBW_BUILD: "cp310-* cp311-* cp312-* cp313-*"
```

### Python Version Identifiers

| Identifier | Python Version |
| ---------- | -------------- |
| `cp310-*`  | Python 3.10    |
| `cp311-*`  | Python 3.11    |
| `cp312-*`  | Python 3.12    |
| `cp313-*`  | Python 3.13    |
| `cp314-*`  | Python 3.14    |
| `cp39-*`   | Python 3.9 (not tested with current nuwa-build) |
| `cp314t-*` | Python 3.14 free-threaded (not tested) |

The generated `nuwa new` workflow builds `cp310-*` through `cp314-*` only.

## Skip Specific Builds

Use `CIBW_SKIP` to exclude certain builds:

```yaml
env:
  # Official Nim binaries are glibc-based, so skip musllinux
  CIBW_SKIP: "*-musllinux_*"

  # Skip Windows 32-bit
  CIBW_SKIP: "win32 *-win32"

  # Skip specific Python versions
  CIBW_SKIP: "cp39-*"
```

## Architecture Targets

### macOS Architectures

Use a native runner for each architecture. Cross-compiling or universal2 wheels
are not in the tested matrix:

```yaml
matrix:
  include:
    - {os: macos-14, arch: arm64}
    - {os: macos-15-intel, arch: x86_64}
```

### Linux Architectures

The action installs Nim matching the native Linux runner:

```yaml
matrix:
  include:
    - {os: ubuntu-latest, arch: x86_64}
    - {os: ubuntu-24.04-arm, arch: aarch64}
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

On Linux, the action installs Nim first and then runs a caller-provided
`CIBW_BEFORE_ALL_LINUX` hook.

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
          nim-version: "2.2.10"
          cibw-version: "4.2.0"
        env:
          # Python 3.10-3.13 only
          CIBW_BUILD: "cp310-* cp311-* cp312-* cp313-*"
          # Official Nim binaries are glibc-based
          CIBW_SKIP: "*-musllinux_*"
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
