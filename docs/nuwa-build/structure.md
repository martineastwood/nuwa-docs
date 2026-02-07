# Project Structure

## Default Layout

Nuwa uses a simple flat layout for easy development:

```
my_project/
├── pyproject.toml           # Python project config
├── nim/                     # Nim source files
│   ├── my_project_lib.nim  # Main entry point (filename = module name)
│   └── helpers.nim          # Additional modules
├── my_project/              # Python package
│   ├── __init__.py          # Package wrapper
│   └── my_project_lib.so    # Compiled extension (generated)
├── tests/                   # Test files
│   └── test_my_project.py   # Pytest tests
├── example.py               # Example/test file
└── README.md
```

## Key Concepts

### Entry Point

The entry point filename determines the Python module name. If your entry point is `my_project_lib.nim`, the compiled extension will be importable as `my_project_lib`.

### Flat Layout

Compiled extensions are placed directly in the package directory. No `pip install -e .` needed - you can run `python example.py` and `pytest` directly after compiling.

### Naming Convention

The compiled extension is named `{module_name}_lib.so` (or `.pyd` on Windows) to avoid conflicts with the Python package directory.

## Multi-File Projects

Use `include` to add code from other Nim files:

**nim/my_project_lib.nim:**
```nim
import nuwa_sdk  # Provides nuwa_export for automatic type stub generation
include helpers  # Include helpers.nim

proc greet(name: string): string {.nuwa_export.} =
  return make_greeting(name)
```

**nim/helpers.nim:**
```nim
proc make_greeting(name: string): string =
  return "Hello, " & name & "!"
```

!!! important
    Use `include` (not `import`) when building shared libraries. The `include` directive includes the code at compile time, while `import` creates a separate module namespace.

## Exporting Functions

You must add the `{.nuwa_export.}` pragma to export functions to Python:

```nim
# ✅ Exported - accessible from Python
proc add(a: int, b: int): int {.nuwa_export.} =
  return a + b

# ❌ Not exported - not accessible from Python
proc subtract(a: int, b: int): int =
  return a - b
```

The `{.nuwa_export.}` pragma:
1. Makes the function callable from Python (via nimpy)
2. Generates type stub information for IDE autocomplete

## Mixing Python and Nim

Your `__init__.py` can import from the compiled extension and add Python wrappers:

```python
# my_project/__init__.py
from .my_project_lib import *

__version__ = "0.1.0"

# Wrap Nim functions with Python code
def validate_dataframe(df, column_name):
    """Extract pandas data and pass to Nim"""
    import numpy as np
    from ctypes import c_void_p

    # Zero-copy numpy view
    data = df[column_name].to_numpy()

    # Pass pointer to Nim
    result = validate_array_raw(
        data.ctypes.data_as(c_void_p),
        len(data)
    )
    return result
```

This allows you to:
- Use Python to extract/prepare data
- Pass pointers/arrays to Nim for zero-copy processing
- Return results back to Python
