# FAQ

Common questions and issues when using nuwa-build-action.

## General

### Can I run custom commands before the build?

Yes, but be careful on Linux. On Linux, this action sets the `CIBW_BEFORE_ALL_LINUX` environment variable to install Nim. If you overwrite this variable, Nim installation will be skipped.

**Safe approach** - Use `CIBW_BEFORE_BUILD`:

```yaml
env:
  # Safe: runs after Nim is installed
  CIBW_BEFORE_BUILD_LINUX: "nimble install deps"
  CIBW_BEFORE_BUILD_MACOS: "nimble install deps"
  CIBW_BEFORE_BUILD_WINDOWS: "nimble install deps"
```

**For macOS and Windows only** - Use standard steps:

```yaml
steps:
  - name: Install dependencies
    run: brew install libffi  # macOS only
    if: runner.os == 'macOS'

  - name: Build wheels
    uses: martineastwood/nuwa-build-action@v1
```

### Where are the wheels stored?

By default, `cibuildwheel` outputs built wheels to the `./wheelhouse/` directory.

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: wheels
    path: ./wheelhouse/*.whl
```

### How do I build specific Python versions?

Use the `CIBW_BUILD` environment variable:

```yaml
env:
  CIBW_BUILD: "cp310-* cp311-* cp312-* cp313-*"
```

See [Advanced Configuration](advanced.md) for details.

### How do I skip certain builds?

Use the `CIBW_SKIP` environment variable:

```yaml
env:
  CIBW_SKIP: "pp* *-musllinux_* win32"
```

## Platform-Specific

### Linux

#### Why does the build fail on Linux?

Linux builds run inside Docker containers. If you're overriding `CIBW_BEFORE_ALL_LINUX`, Nim won't be installed.

**Solution**: Use `CIBW_BEFORE_BUILD` instead:

```yaml
env:
  # Don't override CIBW_BEFORE_ALL_LINUX
  CIBW_BEFORE_BUILD_LINUX: "your commands here"
```

#### How do I install system libraries on Linux?

```yaml
env:
  CIBW_BEFORE_BUILD_LINUX: "yum install -y libffi-devel && nimble install deps"
```

### macOS

#### How do I build for Apple Silicon?

Use the `macos-14` runner which is arm64:

```yaml
strategy:
  matrix:
    os: [macos-13, macos-14]  # Intel and Apple Silicon
```

Or build universal wheels:

```yaml
env:
  CIBW_ARCHS_MACOS: "x86_64 arm64"
```

#### Why does the build fail with "command not found"?

Make sure you're using official GitHub Actions runners. Self-hosted runners may not have the required tools.

### Windows

#### Why does Nim not get found on Windows?

The Chocolatey installation may need a refresh. Add a step to refresh environment variables:

```yaml
- name: Refresh environment
  run: $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")

- name: Build wheels
  uses: martineastwood/nuwa-build-action@v1
```

## Publishing

### How do I publish to PyPI automatically?

See the [Publishing guide](publishing.md) for complete instructions on setting up trusted publishing.

### Can I publish to TestPyPI first?

Yes, use the `repository-url` parameter:

```yaml
- name: Publish to TestPyPI
  uses: pypa/gh-action-pypi-publish@release/v1
  with:
    repository-url: https://test.pypi.org/legacy/
```

### How do I skip publishing on test runs?

Use conditional publishing:

```yaml
publish:
  if: startsWith(github.ref, 'refs/tags/v')
  steps:
    - name: Publish
      uses: pypa/gh-action-pypi-publish@release/v1
```

## Troubleshooting

### Build fails with "Nim compiler not found"

1. Check that you're using the correct action version
2. Verify you're not overriding `CIBW_BEFORE_ALL_LINUX` on Linux
3. Check the build logs for installation errors

### Build succeeds but tests fail

Use `CIBW_TEST_COMMAND` to run tests:

```yaml
env:
  CIBW_TEST_COMMAND: "pytest {project}/tests"
  CIBW_TEST_REQUIRES: "pytest"
```

### Wheels are too large

1. Use release mode: Add `-d:release` to Nim flags in your `pyproject.toml`
2. Skip unnecessary Python versions
3. Use `CIBW_SKIP` for unnecessary architectures

### Build takes too long

1. Reduce the number of platforms/versions:
   ```yaml
   env:
     CIBW_BUILD: "cp311-* cp312-*"
   ```
2. Use build caching
3. Skip musllinux builds:
   ```yaml
   env:
     CIBW_SKIP: "*-musllinux_*"
   ```

## Getting Help

If you're still having issues:

1. Check the [cibuildwheel documentation](https://cibuildwheel.pypa.io/en/stable/)
2. Review the [nuwa-build-action issues](https://github.com/martineastwood/nuwa-build-action/issues)
3. Check the [Nuwa Build documentation](../nuwa-build/)
