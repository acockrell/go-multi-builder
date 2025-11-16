# Platform Support

Complete list of supported operating systems and architectures for cross-compilation with `go-multi-builder`.

## Table of Contents

- [Overview](#overview)
- [Supported Platforms](#supported-platforms)
- [Platform Categories](#platform-categories)
- [Platform-Specific Notes](#platform-specific-notes)
- [Testing Recommendations](#testing-recommendations)
- [Common Platform Sets](#common-platform-sets)

---

## Overview

Go supports cross-compilation to many operating systems and CPU architectures. The `go-multi-builder` script can build for any combination supported by your Go installation.

**Format**: Platforms are specified as `GOOS/GOARCH` pairs, where:
- `GOOS` - Target operating system
- `GOARCH` - Target CPU architecture

**Example**: `linux/amd64` means Linux on x86-64 (64-bit Intel/AMD processors)

---

## Supported Platforms

The following table shows all platform combinations supported by Go:

| GOOS | GOARCH | Description |
|------|--------|-------------|
| **android** | arm | Android on ARM |
| **android** | arm64 | Android on ARM64 |
| **android** | 386 | Android on x86 |
| **android** | amd64 | Android on x86-64 |
| **darwin** | amd64 | macOS on Intel |
| **darwin** | arm64 | macOS on Apple Silicon (M1/M2/M3) |
| **dragonfly** | amd64 | DragonFly BSD on x86-64 |
| **freebsd** | 386 | FreeBSD on x86 (32-bit) |
| **freebsd** | amd64 | FreeBSD on x86-64 |
| **freebsd** | arm | FreeBSD on ARM (32-bit) |
| **freebsd** | arm64 | FreeBSD on ARM64 |
| **illumos** | amd64 | illumos on x86-64 |
| **js** | wasm | WebAssembly |
| **linux** | 386 | Linux on x86 (32-bit) |
| **linux** | amd64 | Linux on x86-64 |
| **linux** | arm | Linux on ARM (32-bit) |
| **linux** | arm64 | Linux on ARM64 (64-bit) |
| **linux** | loong64 | Linux on LoongArch 64-bit |
| **linux** | mips | Linux on MIPS (32-bit, big-endian) |
| **linux** | mipsle | Linux on MIPS (32-bit, little-endian) |
| **linux** | mips64 | Linux on MIPS64 (big-endian) |
| **linux** | mips64le | Linux on MIPS64 (little-endian) |
| **linux** | ppc64 | Linux on PowerPC 64-bit (big-endian) |
| **linux** | ppc64le | Linux on PowerPC 64-bit (little-endian) |
| **linux** | riscv64 | Linux on RISC-V 64-bit |
| **linux** | s390x | Linux on IBM System z |
| **netbsd** | 386 | NetBSD on x86 (32-bit) |
| **netbsd** | amd64 | NetBSD on x86-64 |
| **netbsd** | arm | NetBSD on ARM (32-bit) |
| **netbsd** | arm64 | NetBSD on ARM64 |
| **openbsd** | 386 | OpenBSD on x86 (32-bit) |
| **openbsd** | amd64 | OpenBSD on x86-64 |
| **openbsd** | arm | OpenBSD on ARM (32-bit) |
| **openbsd** | arm64 | OpenBSD on ARM64 |
| **plan9** | 386 | Plan 9 on x86 (32-bit) |
| **plan9** | amd64 | Plan 9 on x86-64 |
| **plan9** | arm | Plan 9 on ARM |
| **solaris** | amd64 | Solaris on x86-64 |
| **windows** | 386 | Windows on x86 (32-bit) |
| **windows** | amd64 | Windows on x86-64 |
| **windows** | arm | Windows on ARM (32-bit) |
| **windows** | arm64 | Windows on ARM64 |

---

## Platform Categories

### Desktop/Workstation

Most common platforms for desktop applications:

```bash
PLATFORMS="linux/amd64,darwin/amd64,darwin/arm64,windows/amd64"
```

- **linux/amd64** - Desktop Linux (Ubuntu, Fedora, etc.)
- **darwin/amd64** - macOS on Intel Macs
- **darwin/arm64** - macOS on Apple Silicon (M1/M2/M3)
- **windows/amd64** - Windows 10/11 (64-bit)

### Server/Cloud

Common platforms for server deployments:

```bash
PLATFORMS="linux/amd64,linux/arm64"
```

- **linux/amd64** - Most x86-64 servers and VMs
- **linux/arm64** - AWS Graviton, Raspberry Pi, ARM servers

### Mobile

Mobile platforms (requires additional setup):

```bash
PLATFORMS="android/arm,android/arm64"
```

- **android/arm** - Older Android devices
- **android/arm64** - Modern Android devices

**Note**: Building for Android requires the Android NDK

### Embedded/IoT

ARM-based embedded systems:

```bash
PLATFORMS="linux/arm,linux/arm64"
```

- **linux/arm** - Raspberry Pi (older), ARM embedded Linux
- **linux/arm64** - Raspberry Pi 3/4/5, ARM64 embedded

### Specialized Architectures

Less common architectures for specific use cases:

```bash
# MIPS (routers, embedded)
PLATFORMS="linux/mips,linux/mipsle,linux/mips64,linux/mips64le"

# PowerPC (IBM servers)
PLATFORMS="linux/ppc64,linux/ppc64le"

# RISC-V (emerging architecture)
PLATFORMS="linux/riscv64"

# IBM System z (mainframes)
PLATFORMS="linux/s390x"
```

### BSD Systems

BSD Unix variants:

```bash
PLATFORMS="freebsd/amd64,openbsd/amd64,netbsd/amd64,dragonfly/amd64"
```

---

## Platform-Specific Notes

### Windows

**File Extension**: Windows binaries automatically get `.exe` extension

```bash
# Outputs: myapp-windows-amd64.exe
./go-multi-build -p "windows/amd64"
```

**32-bit vs 64-bit**:
- `windows/386` - 32-bit Windows (legacy)
- `windows/amd64` - 64-bit Windows (recommended)
- `windows/arm64` - Windows on ARM (Surface Pro X, etc.)

### macOS (Darwin)

**Intel vs Apple Silicon**:
- `darwin/amd64` - Intel Macs (2006-2020)
- `darwin/arm64` - Apple Silicon (M1/M2/M3, 2020+)

**Compatibility**: `darwin/amd64` binaries work on Apple Silicon via Rosetta 2

**Universal Binaries**: Not supported directly (requires `lipo` tool post-build)

### Linux

**Most Portable Platform**: Works across many distributions

**Common Variants**:
- `linux/amd64` - Standard x86-64 servers and desktops
- `linux/386` - 32-bit x86 (legacy, embedded)
- `linux/arm` - 32-bit ARM (Raspberry Pi 1/2, older embedded)
- `linux/arm64` - 64-bit ARM (Raspberry Pi 3+, AWS Graviton, servers)

**glibc vs musl**: Consider static linking for portability:
```bash
./go-multi-build -l '-extldflags "-static"'
```

### Android

**Requires Android NDK**: Cross-compilation needs additional setup

**Variants**:
- `android/arm` - 32-bit ARM (older devices)
- `android/arm64` - 64-bit ARM (modern devices, required for Play Store)
- `android/386` - x86 emulators
- `android/amd64` - x86-64 emulators

### WebAssembly

**Special Platform**: `js/wasm`

```bash
./go-multi-build -p "js/wasm"
# Outputs: myapp-js-wasm
```

**Usage**: For running Go in web browsers

### BSD Systems

**FreeBSD**: Most popular BSD variant
- `freebsd/amd64` - Recommended
- `freebsd/arm64` - ARM servers

**OpenBSD**: Security-focused BSD
- `openbsd/amd64` - Recommended

**NetBSD**: Portable BSD
- `netbsd/amd64` - Recommended

**DragonFly BSD**:
- `dragonfly/amd64` - Only supported architecture

---

## Testing Recommendations

### Minimal Testing Set

Test on these platforms for broad coverage:

```bash
PLATFORMS="linux/amd64,darwin/arm64,windows/amd64"
```

### Comprehensive Testing Set

Test on all platforms you plan to support:

```bash
PLATFORMS="linux/amd64,linux/arm64,darwin/amd64,darwin/arm64,windows/amd64,windows/arm64"
```

### Testing Methods

1. **Native Testing**: Run binary on actual target platform
   ```bash
   # On target system
   ./myapp-linux-amd64
   ```

2. **VM Testing**: Use virtual machines (VirtualBox, VMware, etc.)

3. **Docker Testing**: Test Linux binaries in containers
   ```bash
   docker run --rm -v $(pwd):/app alpine /app/myapp-linux-amd64
   ```

4. **Cross-Platform CI**: Use GitHub Actions, GitLab CI, etc.
   ```yaml
   strategy:
     matrix:
       os: [ubuntu-latest, macos-latest, windows-latest]
   ```

---

## Common Platform Sets

### Minimal (Single Platform)

Current platform only:

```bash
./go-multi-build -p "$(go env GOOS)/$(go env GOARCH)"
```

### Standard (Desktop)

Most common desktop platforms:

```bash
PLATFORMS="linux/amd64,darwin/amd64,darwin/arm64,windows/amd64"
```

### Extended Desktop

Desktop plus 32-bit Windows:

```bash
PLATFORMS="linux/amd64,darwin/amd64,darwin/arm64,windows/386,windows/amd64"
```

### Server Optimized

Cloud and server deployments:

```bash
PLATFORMS="linux/amd64,linux/arm64"
```

### Complete Cross-Platform

All major platforms:

```bash
PLATFORMS="linux/386,linux/amd64,linux/arm,linux/arm64,darwin/amd64,darwin/arm64,windows/386,windows/amd64,windows/arm64"
```

### ARM-Focused

ARM servers and embedded:

```bash
PLATFORMS="linux/arm,linux/arm64,darwin/arm64,windows/arm64"
```

### Legacy Support

Including older 32-bit systems:

```bash
PLATFORMS="linux/386,linux/amd64,windows/386,windows/amd64,freebsd/386,freebsd/amd64"
```

---

## Checking Supported Platforms

List all platforms supported by your Go installation:

```bash
go tool dist list
```

Check specific platform support:

```bash
go tool dist list | grep linux
go tool dist list | grep darwin
go tool dist list | grep windows
```

---

## CGO Limitations

**Important**: `go-multi-builder` sets `CGO_ENABLED=0` for cross-compilation.

**Limitations**:
- No C library dependencies
- No SQLite (unless using pure-Go driver)
- No certain crypto implementations requiring C

**Workarounds**:
1. Use pure Go libraries
2. Build natively on each platform
3. Use Docker with cross-compilation toolchains

---

## Architecture Aliases

Some architectures have aliases:

- `amd64` = `x86-64` = `x64`
- `386` = `x86` = `i386`
- `arm64` = `aarch64`

Use the Go standard names (amd64, 386, arm64) for consistency.

---

For configuration options, see [configuration.md](./configuration.md).
For command-line flags, see [commands.md](./commands.md).
