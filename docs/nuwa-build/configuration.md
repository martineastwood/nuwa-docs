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
nimble-deps = ["nimpy", "cligen >= 1.0.0"]
```

## Configuration Options

| Option            | Type   | Default                   | Description                                                    |
| ----------------- | ------ | ------------------------- | -------------------------------------------------------------- |
| `nim-source`      | string | `"nim"`                   | Directory containing Nim source files                          |
| `module-name`     | string | Derived from project name | Python package name                                            |
| `lib-name`        | string | `{module_name}_lib`       | Internal compiled extension name                               |
| `entry-point`     | string | `{lib_name}.nim`          | Main entry point file (relative to `nim-source`)               |
| `output-location` | string | `"auto"`                  | Where to place compiled extension (`"auto"`, `"src"`, or path) |
| `nim-flags`       | list   | `[]`                      | Additional compiler flags                                      |
| `nimble-deps`     | list   | `[]`                      | Nimble packages to auto-install before build                   |

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
