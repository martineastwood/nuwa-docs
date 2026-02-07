# Nuwa Build

Zero-configuration build system for compiling Nim code into Python extensions.

## Overview

Nuwa Build is a zero-configuration build system for compiling Nim code into Python extensions with sensible defaults and minimal configuration.

### Key Features

- **Zero Configuration** - Works out of the box
- **Multi-file Projects** - Compile multiple Nim files into a single extension
- **Flexible Configuration** - Configure via `pyproject.toml` or CLI
- **PEP 517/660 Compatible** - Build wheels and source distributions
- **Cross-Platform CI** - GitHub Actions with manylinux support
- **Editable Installs** - `pip install -e .` support
- **Watch Mode** - Auto-recompile on file changes
- **Auto Dependencies** - Automatically install Nimble packages
- **Testing Support** - Includes pytest in project template
- **Jupyter Support** - Compile Nim code directly in notebooks

### Requirements

- **Python**: 3.9+ (including Python 3.14 with free-threaded ABI)
- **Nim**: Compiler must be installed and in PATH
- **nimpy**: Install via `nimble install nimpy`

## Installation

```bash
# Install Nuwa Build
pip install nuwa-build

# Install nimpy (Nim-Python bridge)
nimble install nimpy
```

## Quick Start

```bash
# Create a new project
nuwa new my_project
cd my_project

# Compile (debug build)
nuwa develop

# Compile (release build)
nuwa develop --release

# Run the example
python example.py

# Run tests
pytest
```

## Next Steps

- [Project Structure](structure.md) - Learn about the project layout
- [Configuration](configuration.md) - Configure your build
- [CLI Commands](commands.md) - Command reference
- [Jupyter Support](jupyter.md) - Use in notebooks
- [AI Agents](ai-agents.md) - Setup AI coding assistants
- [CI/CD](ci.md) - GitHub Actions setup
- [Troubleshooting](troubleshooting.md) - Common issues
