# Configuration

## pyproject.toml

Configure your project in the `[tool.nuwa]` section:

```toml
[build-system]
requires = ["nuwa-build"]
build-backend = "nuwa_build"

[project]
name = "my-package"
version = "0.1.0"

[tool.nuwa]
# Nim source directory (default: "nim")
nim-source = "nim"

# Python module name (default: derived from project name)
module-name = "my_package"

# Internal library name (default: "{module_name}_lib")
lib-name = "my_package_lib"

# Entry point file (default: "{lib_name}.nim")
entry-point = "my_package_lib.nim"

# Output location: "auto", "src", or explicit path
output-location = "auto"

# Additional Nim compiler flags
nim-flags = []

# Nimble dependencies (auto-installed before build)
nimble-deps = ["nimpy@0.2.1", "nuwa_sdk@0.4.3"]
```

## Configuration Options

| Option                  | Type   | Default                   | Description                                                    |
| ----------------------- | ------ | ------------------------- | -------------------------------------------------------------- |
| `nim-source`            | string | `"nim"`                   | Directory containing Nim source files                          |
| `module-name`           | string | Derived from project name | Python package name                                            |
| `lib-name`              | string | `{module_name}_lib`       | Internal compiled extension name                               |
| `entry-point`           | string | `{lib_name}.nim`          | Main entry point file (relative to `nim-source`)               |
| `output-location`       | string | `"auto"`                  | Where to place compiled extension (`"auto"`, `"src"`, or path) |
| `nim-flags`             | list   | `[]`                      | Additional compiler flags                                      |
| `nimble-deps`           | list   | `[]`                      | Nimble packages to auto-install before build                   |
| `windows-static-linking` | bool   | `true`                    | Enable static linking on Windows (see below)                  |
| `bundle-adjacent-dlls`  | bool   | `true`                    | Bundle adjacent DLL files into wheel (see below)              |

## Windows Static Linking

On Windows, Nuwa automatically enables static linking by default. This means the compiled extension (`.pyd`) will be statically linked against system libraries, avoiding DLL dependencies.

### How It Works

When `windows-static-linking` is `true` (the default):
- On Windows only, adds `--passL:-static` to the Nim compiler flags
- This applies when using the GCC-based compiler (not `--cc:vcc`)
- Results in a self-contained `.pyd` file with no runtime DLL dependencies

### Disabling Static Linking

If you need to disable static linking on Windows:

```toml
[tool.nuwa]
windows-static-linking = false
```

### DLL Bundling (`bundle-adjacent-dlls`)

When static linking is disabled, DLL dependencies may be generated. The `bundle-adjacent-dlls` option controls whether these DLLs are bundled into the wheel.

**When `bundle-adjacent-dlls` is `true`** (default):

- Any DLL files found next to the compiled `.pyd` are bundled into the wheel
- DLLs are placed in the same package directory as the `.pyd`
- The extension will work without requiring external DLL installation

**When `bundle-adjacent-dlls` is `false`**:

- DLLs are NOT bundled into the wheel
- End-users must have the required DLLs installed on their system
- Only use this if you're distributing DLLs separately

### Interaction Between Options

| `windows-static-linking` | `bundle-adjacent-dlls` | Result |
| ------------------------ | ---------------------- | ------ |
| `true` (default) | `true` (default) | Static linking, no DLLs generated (recommended) |
| `true` | `false` | Same as above (no DLLs to bundle) |
| `false` | `true` | DLLs generated AND bundled into wheel |
| `false` | `false` | DLLs generated, NOT bundled (manual distribution required) |

!!! note
    Static linking is the default and recommended approach for Windows builds. It produces standalone wheels that are easier to distribute and install.

## Build Profiles

Build profiles allow you to define preset compiler flag configurations:

```toml
[tool.nuwa.profiles.dev]
nim-flags = ["-d:debug", "--debugger:native", "--linenos:on"]

[tool.nuwa.profiles.release]
nim-flags = ["-d:release", "--opt:speed", "--stacktrace:off"]

[tool.nuwa.profiles.bench]
nim-flags = ["-d:release", "--opt:speed", "--stacktrace:on"]

[tool.nuwa.profiles.size]
nim-flags = ["-d:release", "--opt:size"]
```

**Usage:**
```bash
nuwa develop --profile dev
nuwa build --profile release
nuwa watch --profile bench
```

**Flag precedence:**

1. Base `nim-flags` from `[tool.nuwa]`
2. Profile flags (appended)
3. CLI `--nim-flag` arguments (applied last)

## Output Location

The `output-location` setting controls where the compiled extension is placed:

| Value       | Description                                      |
| ----------- | ------------------------------------------------ |
| `"auto"`    | Flat layout - places extension in `{module_name}/` |
| `"src"`     | Uses `src/{module_name}/` (for old projects)     |
| Explicit path | Use a custom output directory                   |

## Package Data

Nuwa automatically includes all non-Python files in your package directory when building wheels.

### Automatic Inclusion

All files are included except:
- Python cache (`__pycache__`, `*.pyc`, `*.pyo`)
- Compiled extensions (`.so`, `.pyd`, `.dll`)
- Version control (`.git`, `.hg`, `.svn`)
- Build artifacts (`dist/`, `build/`, `*.egg-info`)
- Development caches (`.pytest_cache`, `.mypy_cache`, `.ruff_cache`)
- IDE files (`.vscode`, `.idea`, `.DS_Store`)
- Test directories (`tests/`, `test/`)

### Fine-Grained Control

For precise control, create a `MANIFEST.in` file:

```
include package/config.json
recursive-include package/templates *.html *.css *.js
global-include *.md
exclude package/dev_config.yaml
recursive-exclude package/tests *.py
```

**Supported commands:**

- `include pattern ...` - Include files matching patterns
- `exclude pattern ...` - Exclude files matching patterns
- `recursive-include dir pattern ...` - Include files in directory
- `recursive-exclude dir pattern ...` - Exclude files in directory
- `global-include pattern ...` - Include files anywhere
- `global-exclude pattern ...` - Exclude files anywhere
