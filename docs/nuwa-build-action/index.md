# Nuwa Build Action

A GitHub Action to automatically build Python wheels for **[Nuwa Build](../nuwa-build/)** projects using `cibuildwheel`.

## Overview

This action handles the complexity of installing the Nim compiler across different platforms (Linux Docker containers, macOS, and Windows) and orchestrates the wheel build process.

### Features

- **Zero Configuration** - Automatically sets up Nim on the runner or inside build containers
- **Multi-Platform Support** - Builds wheels for Linux (`manylinux`), macOS (`x86_64` & `arm64`), and Windows
- **Cibuildwheel Integration** - Leverages industry-standard `cibuildwheel` for reliable wheel generation
- **Customizable** - Supports specific Nim versions and standard cibuildwheel environment variables

## Quick Start

### Basic Workflow

```yaml
name: Build Wheels

on: [push, pull_request]

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

      - uses: actions/upload-artifact@v4
        with:
          name: cibw-wheels-${{ matrix.os }}-${{ strategy.job-index }}
          path: ./wheelhouse/*.whl
```

## Inputs

| Input          | Description                                 | Default  |
| -------------- | ------------------------------------------- | -------- |
| `nim-version`  | The version of the Nim compiler to install | `2.0.0`  |
| `cibw-version` | The version of `cibuildwheel` to use       | `2.22.0` |

## Next Steps

- [Advanced Configuration](advanced.md) - Control Python versions, architectures, and more
- [Platform Details](platforms.md) - How Nim is installed on each platform
- [Publishing to PyPI](publishing.md) - Automated PyPI publishing
- [FAQ](faq.md) - Common questions and issues
