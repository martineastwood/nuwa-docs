# CLI Commands

## `nuwa new <path>`

Create a new project scaffold:

```bash
nuwa new my_project
nuwa new my_project --name custom-name
```

## `nuwa init [path]`

Initialize Nuwa in an existing project:

```bash
# Initialize in current directory
nuwa init

# Initialize in specific directory
nuwa init /path/to/project
```

This command:
- Adds `[build-system]` and `[tool.nuwa]` to `pyproject.toml`
- Creates `nim/` directory with entry point and helper files
- Updates `.gitignore` with build artifacts
- Non-destructive: won't overwrite existing files

## `nuwa develop`

Compile the project in-place:

```bash
# Debug build
nuwa develop

# Release build
nuwa develop --release

# Use build profile
nuwa develop --profile dev

# Override configuration
nuwa develop --module-name my_module
nuwa develop --nim-source my_nim_dir
nuwa develop --entry-point main.nim
nuwa develop --nim-flag="-d:danger"
```

After running `nuwa develop`, the compiled extension is in `{module_name}/`. You can run `python example.py` or `pytest` directly without any installation step.

## `nuwa watch`

Watch for file changes and automatically recompile:

```bash
# Watch for changes and auto-recompile
nuwa watch

# Watch with tests after each compile
nuwa watch --run-tests

# Watch in release mode
nuwa watch --release

# Watch with build profile
nuwa watch --profile bench
```

## `nuwa build`

Build a wheel package for distribution:

```bash
# Build a wheel (output: dist/)
nuwa build

# Build with custom Nim flags
nuwa build --nim-flag="-d:release"

# Build with profile
nuwa build --profile release

# Override configuration
nuwa build --module-name my_module
```

## `nuwa clean`

Clean build artifacts and dependencies:

```bash
# Clean everything (dependencies + artifacts)
nuwa clean

# Clean only dependencies (.nimble/ directory)
nuwa clean --deps

# Clean only build artifacts
nuwa clean --artifacts
```

**What gets cleaned:**
- `--deps`: Removes `.nimble/` directory (local Nimble packages)
- `--artifacts`: Removes `nimcache/`, `build/`, `dist/` and compiled `.so`/`.pyd` files
- (no flags): Cleans both

## Shell Completion

Nuwa supports shell completion for bash, zsh, and fish:

```bash
# Install shtab
pip install shtab

# Generate and install completions
nuwa --print-completion bash > ~/.local/share/bash-completion/completions/nuwa
nuwa --print-completion zsh > ~/.zfunc/_nuwa
nuwa --print-completion fish > ~/.config/fish/completions/nuwa.fish
```

For zsh, add to `~/.zshrc`:
```bash
fpath=(~/.zfunc $fpath)
autoload -U compinit && compinit
```
