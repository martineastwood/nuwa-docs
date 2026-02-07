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

```toml
[tool.nuwa]
nimble-deps = ["nimpy"]
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
