# Platform Details

This action installs the Nim compiler so `cibuildwheel` can compile your extension. See the [support matrix](../support.md) for what is tested.

## Linux

Wheels are built inside manylinux Docker containers.

The action:

1. Appends `*-musllinux*` to `CIBW_SKIP` unless you already skipped musllinux (official Nim binaries are glibc).
2. Sets `CIBW_BEFORE_ALL_LINUX` to install `xz`, `curl`, and `gcc`, then download `nim-<version>-linux_x64.tar.xz` from nim-lang.org and verify the `.sha256` file.

**Linux aarch64 is not supported by this installer.** The archive name is always `linux_x64`.

### Container images

cibuildwheel chooses the manylinux image. You can override image names with the usual `CIBW_MANYLINUX_*` variables. That does not change the Nim architecture: it remains x86_64.

### Customizing Linux builds

!!! warning
    If you override `CIBW_BEFORE_ALL_LINUX`, Nim installation will be skipped.

Use `CIBW_BEFORE_BUILD` instead:

```yaml
env:
  CIBW_BEFORE_BUILD_LINUX: "nimble install cligen"
```

## macOS

The action downloads the official archive for the runner CPU:

- `nim-<version>-macosx_arm64.tar.xz` on Apple Silicon
- `nim-<version>-macosx_x64.tar.xz` on Intel

Checksums are verified with `shasum -a 256 -c`. Cross-compiling a macOS wheel for the other architecture is not in the tested matrix.

```yaml
steps:
  - name: Build wheels (Apple Silicon runner)
    runs-on: macos-14
    # Native arm64 Nim
```

## Windows

- Installs the requested Nim version with Chocolatey
- Installs MinGW so a C compiler is present
- Adds `C:\tools\Nim\nim-<version>\bin` and MinGW to `PATH`

Default generated projects statically link MinGW runtimes via nuwa-build (`windows-static-linking = true`).

## Environment variables

Prefer `CIBW_BEFORE_BUILD_*` for extra setup. Do not replace `CIBW_BEFORE_ALL_LINUX` unless you install Nim yourself.

See [cibuildwheel settings](https://cibuildwheel.pypa.io/en/stable/options/) for `CIBW_BUILD`, `CIBW_SKIP`, and architecture flags.
