# Troubleshooting

## "Nim compiler not found"

Make sure Nim is installed and in your PATH:

```bash
nim --version
```

Install from https://nim-lang.org/install.html if needed.

## "cannot open file: nimpy"

You need to install the nimpy library.

**Option 1: Auto-install via configuration** (Recommended)

New projects already list `nimpy` and `nuwa_sdk` in `nimble-deps`. `nuwa develop` and `nuwa build` install them.

```toml
[tool.nuwa]
nimble-deps = ["nimpy@0.2.1", "nuwa_sdk@0.4.4"]
```

**Option 2: Manual installation**

```bash
nimble install nimpy
```

## "nimble package manager not found"

Nimble is installed with Nim. Make sure Nim is properly installed and in your PATH:

```bash
nim --version
nimble --version
```

If nimble is not found, reinstall Nim from https://nim-lang.org/install.html.

## "ModuleNotFoundError: No module named 'my_package'"

The module needs to be compiled first. Run:

```bash
nuwa develop
```

Then you can import it directly from the project root. No `pip install` needed!

For pytest: Make sure you've compiled the extension with `nuwa develop` first.

## "ValueError: Module name '...' is not a valid Python identifier"

Your project name contains invalid characters for Python modules. Module names can only contain letters, numbers, and underscores, and cannot start with a number. Use the `--name` option:

```bash
nuwa new my-project --name my_valid_name
```

## "Multiple .nim files found in nim/"

Nuwa found multiple `.nim` files but can't determine which is the entry point. Specify it in `pyproject.toml`:

```toml
[tool.nuwa]
entry-point = "my_entry_file.nim"
```

Or ensure there's only one `.nim` file, or name your entry point `{module_name}_lib.nim`.

## Entry Point Discovery

If `entry-point` is not specified, Nuwa will automatically discover the main entry point using this priority:

1. Explicit `[tool.nuwa] entry-point` configuration
2. `{module_name}_lib.nim` (matches the lib-name)
3. `lib.nim` (fallback convention)
4. First (and only) `.nim` file if only one exists
5. Error if multiple files found and no clear entry point

## Windows-Specific Issues

### DLL Not Found Errors

If you get "DLL not found" errors when importing your extension on Windows:

1. **Ensure static linking is enabled** (recommended):
   ```toml
   [tool.nuwa]
   windows-static-linking = true
   ```

2. **Rebuild with static linking**:
   ```bash
   nuwa develop --release
   ```

!!! warning
    If you disable `windows-static-linking`, make sure `bundle-adjacent-dlls` is enabled (the default) to have DLLs automatically bundled into the wheel. Otherwise, you must ensure those DLLs are available on the target system, or distribute them separately.

### MSVC vs GCC Compiler Issues

On Windows, Nim can use either MSVC (`--cc:vcc`) or MinGW GCC (`--cc:gcc`). The static linking feature only works with GCC.

If you encounter issues with the default compiler:

```toml
[tool.nuwa]
# Explicitly use GCC for static linking support
nim-flags = ["--cc:gcc"]
```

Or to use MSVC (static linking will be skipped):

```toml
[tool.nuwa]
nim-flags = ["--cc:vcc"]
windows-static-linking = false  # Not applicable for MSVC
```

### "Cannot open source file: stdio.h"

This error indicates the Windows C++ build tools are not properly installed. Install the Microsoft C++ Build Tools:

1. Download from https://visualstudio.microsoft.com/visual-cpp-build-tools/
2. Install "Desktop development with C++" workload
3. Restart your terminal and retry the build
