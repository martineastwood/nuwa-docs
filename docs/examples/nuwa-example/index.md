# Nuwa Example

Example projects demonstrating how to use Nuwa tools.

## Overview

This repository contains example projects showing how to build Python extensions with Nim using the Nuwa toolchain.

## example_project

A comprehensive example project that demonstrates all the key features of building Python extensions with Nim using Nuwa.

### Key Features Demonstrated

- **Zero-configuration build system** - No setup.py or custom build scripts needed
- **Automatic type stubs** (`.pyi` files) - Full IDE autocomplete and type checking
- **Build profiles** - Debug, release, and benchmarking configurations
- **Editable installs** - Rapid development workflow
- **NumPy integration** - Zero-copy array access using the buffer protocol
- **GitHub Actions CI/CD** - Automated multi-platform wheel building and PyPI publishing
- **pytest integration** - Complete test suite for Nim extensions

### Example Functions

The project includes examples of common Nim-Python interop patterns:

#### Basic Types

- `greet(name: string) -> string` - String input/output
- `add(a: int, b: int) -> int` - Multiple parameters with primitive types
- `is_even(n: int) -> bool` - Boolean return type

#### Error Handling

- `safe_divide(a: float, b: float) -> float` - Raises Python exceptions

#### Collections

- `calculate_average(numbers: seq[float]) -> float` - Process Python sequences

#### Default Parameters

- `power(base: float, exponent: float = 2.0) -> float` - Optional parameters

#### String Operations

- `reverse_string(s: string) -> string` - String manipulation using Nim stdlib
- `count_words(text: string) -> int` - Using Nim's string utilities

#### NumPy Arrays (Zero-Copy)

- `numpy_array_sum(arr) -> int` - Read int64 arrays without copying data
- `numpy_array_multiply_scalar(arr, scalar) -> list[float]` - Process float64 arrays

### Quick Start

```bash
# Clone and navigate to the example
cd nuwa-example/example_project

# Build the extension (debug mode)
nuwa develop

# Run the example script
python example.py

# Run tests
pytest
```

### Project Structure

```
example_project/
├── .github/
│   └── workflows/
│       └── publish.yml          # PyPI publishing workflow
├── nim/                          # Nim source files
│   ├── example_project_lib.nim    # Main entry point
│   └── helpers.nim              # Additional modules
├── example_project/               # Python package
│   ├── __init__.py              # Package wrapper
│   ├── example_project_lib.so   # Compiled extension (generated)
│   └── example_project_lib.pyi  # Type stubs (generated)
├── tests/                       # Pytest tests
│   └── test_example_project.py
├── example.py                   # Usage examples
└── pyproject.toml               # Project configuration
```

### Configuration

The project uses `pyproject.toml` for all configuration:

```toml
[build-system]
requires = ["nuwa-build"]
build-backend = "nuwa_build"

[tool.nuwa]
nim-source = "nim"
module-name = "example_project"
lib-name = "example_project_lib"
entry-point = "example_project_lib.nim"
nimble-deps = ["nimpy@0.2.1", "nuwa_sdk@0.3.0"]

# Build profiles
[tool.nuwa.profiles.dev]
nim-flags = ["-d:debug", "--debugger:native", "--linenos:on"]

[tool.nuwa.profiles.release]
nim-flags = ["-d:release", "--opt:speed"]
```

### Adding New Functions

1. Edit `nim/example_project_lib.nim`:
```nim
import nuwa_sdk

proc my_function(x: int, y: int): string {.nuwa_export.} =
  ## Your function documentation
  return "Result: " & $(x + y)
```

2. Rebuild:
```bash
nuwa develop
```

3. Use from Python (with full autocomplete):
```python
from example_project import my_function

# IDE shows: my_function(x: int, y: int) -> str
result = my_function(5, 10)
```

### Publishing to PyPI

The project includes a GitHub Actions workflow for automated publishing:

```bash
git tag v1.0.0
git push origin v1.0.0
```

The workflow automatically builds wheels for:

- **Platforms**: Linux, macOS, Windows
- **Python versions**: 3.9, 3.10, 3.11, 3.12, 3.13, 3.14

### Source

Repository: [github.com/martineastwood/nuwa-example](https://github.com/martineastwood/nuwa-example)
