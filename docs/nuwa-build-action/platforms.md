# Platform Details

This action installs the Nim compiler so `cibuildwheel` can compile your extension. See the [support matrix](../support.md) for what is tested.

## Linux

Wheels are built inside manylinux Docker containers.

The action:

1. Appends `*-musllinux*` to `CIBW_SKIP` unless you already skipped musllinux (official Nim binaries are glibc).
2. Selects `linux_x64` or `linux_arm64` from the native runner architecture.
3. Prepends installation commands to `CIBW_BEFORE_ALL_LINUX` to install `xz`, `curl`, and `gcc`, download Nim, and verify its `.sha256` file.
4. Runs any caller-provided `CIBW_BEFORE_ALL_LINUX` commands after Nim is available.

Linux aarch64 is tested on the native `ubuntu-24.04-arm` runner. Use a native
runner matching the requested wheel architecture.

### Container images

cibuildwheel chooses the manylinux image. You can override image names with the
usual `CIBW_MANYLINUX_*` variables; the Nim archive continues to match the
runner architecture.

### Customizing Linux builds

The action preserves `CIBW_BEFORE_ALL_LINUX` and runs it after installing Nim:

```yaml
env:
  CIBW_BEFORE_ALL_LINUX: "nimble install cligen -y"
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
- Uses the MinGW dependency installed by the Chocolatey Nim package
- Adds `C:\tools\Nim\nim-<version>\bin` to `PATH`; Chocolatey supplies the compiler path

Default generated projects statically link MinGW runtimes via nuwa-build (`windows-static-linking = true`).

## Environment variables

Use the standard cibuildwheel hooks for extra setup. On Linux, a supplied
`CIBW_BEFORE_ALL_LINUX` hook is appended after Nim installation.

See [cibuildwheel settings](https://cibuildwheel.pypa.io/en/stable/options/) for `CIBW_BUILD`, `CIBW_SKIP`, and architecture flags.
