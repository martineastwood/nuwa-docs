# Nuwa SDK

Python interoperability utilities for building high-performance Python extensions with Nim.

## Overview

Nuwa SDK provides core utilities for Nim/Python interop when building extensions with [Nuwa Build](../nuwa-build/).

### Features

- **`nuwa_export`** - Macro for automatic generation of Python type stubs (`.pyi` files)
- **`withNogil`** - Template for releasing the Python GIL during pure Nim code execution

## Installation

Nuwa SDK is automatically installed when you create a new project with `nuwa new`.

Manual installation:

```bash
nimble install nuwa_sdk
```

## Quick Start

```nim
import nuwa_sdk

proc add(a: int, b: int): int {.nuwa_export.} =
  ## Add two integers together
  return a + b

proc greet(name: string): string {.nuwa_export.} =
  ## Greet a person by name
  return "Hello, " & name
```

When you build with `nuwa build`, the macro:
1. Exports the function to Python (via `nimpy`)
2. Emits compile-time metadata about function signatures
3. Enables automatic generation of `.pyi` stub files

## Next Steps

- [nuwa_export Macro](nuwa-export.md) - Automatic type stub generation
- [GIL Release](gil-release.md) - Parallel execution with withNogil
- [Type Mappings](types.md) - Nim to Python type conversions
