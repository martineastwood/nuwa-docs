# Platform Details

This action performs platform-specific setup to ensure the Nim compiler is available during the build process.

## Linux

### How It Works

On Linux, wheels are built inside Docker containers (usually `manylinux`) to ensure compatibility.

The action:
1. Injects a script into `CIBW_BEFORE_ALL_LINUX`
2. The script installs `gcc` and `curl`
3. Downloads official Nim binaries via `curl`
4. Extracts and adds Nim to PATH inside the Docker container

### Container Environment

```yaml
# manylinux2014 (default)
CIBW_MANYLINUX_X86_64_IMAGE: manylinux2014
CIBW_MANYLINUX_I686_IMAGE: manylinux2014
CIBW_MANYLINUX_AARCH64_IMAGE: manylinux2014
CIBW_MANYLINUX_PPC64LE_IMAGE: manylinux2014
CIBW_MANYLINUX_S390X_IMAGE: manylinux2014

# manylinux_2_28 (optional)
CIBW_MANYLINUX_X86_64_IMAGE: manylinux_2_28
```

### Installation Script

The action uses a script similar to:

```bash
# Inside manylinux container
yum install -y gcc curl

# Download and install Nim
NIM_VERSION="2.0.0"
curl -L -o nim.tar.xz \
  "https://nim-lang.org/download/nim-${NIM_VERSION}-linux_x64.tar.xz"

tar xf nim.tar.xz
mv nim-${NIM_VERSION}-linux_x64 /opt/nim
export PATH="/opt/nim/bin:$PATH"
```

### Customizing Linux Builds

!!! warning
    If you override `CIBW_BEFORE_ALL_LINUX`, Nim installation will be skipped.

Use `CIBW_BEFORE_BUILD` instead:

```yaml
env:
  # Safe: runs after Nim is installed
  CIBW_BEFORE_BUILD_LINUX: "nimble install cligen"
```

## macOS

### How It Works

- Installs Nim using the official `choosenim` installer
- Adds Nim to the system PATH
- Supports both x86_64 (Intel) and arm64 (Apple Silicon)

### Installation Process

```bash
# Install choosenim
curl https://nim-lang.org/choosenim/init.sh -sSf | sh

# Install specific Nim version
choosenim 2.0.0

# Add to PATH
export PATH="$HOME/.nimble/bin:$PATH"
```

### Architecture Specifics

```yaml
steps:
  - name: Build wheels (Intel)
    runs-on: macos-13
    # Uses x86_64 runner

  - name: Build wheels (Apple Silicon)
    runs-on: macos-14
    # Uses arm64 runner
```

### Customizing macOS Builds

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Install system dependencies
    run: brew install libffi

  - name: Build wheels
    uses: martineastwood/nuwa-build-action@v1
```

## Windows

### How It Works

- Installs Nim using `chocolatey` package manager
- Adds Nim to the system PATH
- Requires Windows runners

### Installation Process

```powershell
# Install via Chocolatey
choco install nim -y

# Refresh environment variables
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

### Customizing Windows Builds

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Install system dependencies
    run: choco install openssl -y

  - name: Build wheels
    uses: martineastwood/nuwa-build-action@v1
```

## Platform-Specific Environment Variables

### Linux-only

```yaml
env:
  CIBW_BEFORE_ALL_LINUX: "yum install -y libffi-devel"
  CIBW_BEFORE_BUILD_LINUX: "nimble install deps"
```

### macOS-only

```yaml
env:
  CIBW_BEFORE_ALL_MACOS: "brew install libffi"
  CIBW_BEFORE_BUILD_MACOS: "nimble install deps"
```

### Windows-only

```yaml
env:
  CIBW_BEFORE_ALL_WINDOWS: "choco install openssl"
  CIBW_BEFORE_BUILD_WINDOWS: "nimble install deps"
```

## Cross-Platform Example

```yaml
jobs:
  build_wheels:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        include:
          - os: ubuntu-latest
            cibw-before: "yum install -y libffi-devel"
          - os: macos-latest
            cibw-before: "brew install libffi"
          - os: windows-latest
            cibw-before: "choco install openssl"

    steps:
      - uses: actions/checkout@v4

      - name: Install system dependencies
        run: ${{ matrix.cibw-before }}

      - name: Build wheels
        uses: martineastwood/nuwa-build-action@v1
```
