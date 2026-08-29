# Support matrix

This is what the Nuwa toolchain **tests today**. Anything else may work, but is not a supported claim.

| Item | Status |
| --- | --- |
| CPython 3.10, 3.11, 3.12, 3.13, 3.14 | Tested |
| Python 3.9 | Dropped in nuwa-build 0.5.0 |
| Free-threaded CPython (`cp314t`) | Not tested (nimpy not verified) |
| PyPy | Not tested |
| Linux wheels | manylinux **x86_64** |
| musllinux | Skipped (official Nim binaries are glibc) |
| Linux aarch64 | Not tested (`nuwa-build-action` installs `linux_x64` Nim) |
| macOS | Native Intel or Apple Silicon archive |
| Windows | 64-bit; MinGW runtimes statically linked by default |
| Nim | 2.2.x in CI and the Action default (`2.2.10`) |

Versions for the current line-up:

| Package | Version |
| --- | --- |
| nuwa-build | 0.5.0 |
| nuwa-sdk | 0.4.3 |
| nuwa-build-action | `@v1` (Nim 2.2.10, cibuildwheel 4.2.0) |
