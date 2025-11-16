# go-multi-builder Documentation

Complete guide to building Go projects for multiple platforms.

## Table of Contents

- [Overview](#overview)
- [Getting Started](#getting-started)
  - [Installation](#installation)
  - [Basic Usage](#basic-usage)
  - [Your First Build](#your-first-build)
- [Features](#features)
  - [Parallel Builds](#parallel-builds)
  - [Version Embedding](#version-embedding)
  - [Incremental Builds](#incremental-builds)
  - [Binary Compression](#binary-compression)
  - [Build Summary](#build-summary)
  - [Color Output](#color-output)
- [Configuration](#configuration)
  - [Configuration Priority](#configuration-priority)
  - [Config File](#config-file)
  - [Environment Variables](#environment-variables)
  - [Command-Line Flags](#command-line-flags)
- [Usage Patterns](#usage-patterns)
  - [Development Workflow](#development-workflow)
  - [CI/CD Integration](#cicd-integration)
  - [Release Builds](#release-builds)
- [Advanced Topics](#advanced-topics)
  - [Custom Build Flags](#custom-build-flags)
  - [Cross-Compilation Caveats](#cross-compilation-caveats)
  - [Debugging Build Issues](#debugging-build-issues)
- [Reference Documentation](#reference-documentation)
- [Troubleshooting](#troubleshooting)

---

## Overview

`go-multi-builder` is a Bash script that simplifies building Go applications for multiple operating systems and architectures. It automates cross-compilation, provides parallel builds for speed, and offers comprehensive reporting.

### Key Benefits

- **Time Savings**: Build for multiple platforms in parallel
- **Consistency**: Same build process across all platforms
- **Visibility**: Clear reporting of build status and metrics
- **Flexibility**: Configure via CLI, env vars, or config files
- **Reliability**: Validates environment and provides helpful errors

---

## Getting Started

### Installation

#### Method 1: Direct Download

```bash
curl -O https://raw.githubusercontent.com/acockrell/go-multi-builder/main/go-multi-build
chmod +x go-multi-build
```

#### Method 2: Clone Repository

```bash
git clone https://github.com/acockrell/go-multi-builder.git
cd go-multi-builder
chmod +x go-multi-build
```

#### Method 3: Add to PATH

```bash
# Download to a directory in your PATH
sudo curl -o /usr/local/bin/go-multi-build https://raw.githubusercontent.com/acockrell/go-multi-builder/main/go-multi-build
sudo chmod +x /usr/local/bin/go-multi-build
```

### Basic Usage

The simplest usage requires no arguments if you're in a Go module directory:

```bash
./go-multi-build
```

This will:
1. Auto-detect the package name from `go.mod`
2. Build for default platforms (linux/amd64, darwin/amd64, darwin/arm64)
3. Output binaries to the current directory

### Your First Build

1. Navigate to your Go project:
   ```bash
   cd ~/my-go-project
   ```

2. Run the builder:
   ```bash
   /path/to/go-multi-build
   ```

3. Check the output:
   ```bash
   ls -lh myapp-*
   ```

You should see binaries like:
- `myapp-linux-amd64`
- `myapp-darwin-amd64`
- `myapp-darwin-arm64`

---

## Features

### Parallel Builds

Build multiple platforms simultaneously to save time:

```bash
./go-multi-build -P
```

On a multi-core system, this can reduce build time by 50-70% when building for 3+ platforms.

**When to use:**
- Building for 3+ platforms
- Multi-core development machine
- CI/CD with sufficient resources

**When to avoid:**
- Limited CPU/memory
- Debugging build issues (sequential is clearer)

### Version Embedding

Automatically embed version information from git into your binaries:

```bash
./go-multi-build -v
```

This injects three variables into your `main` package:
- `main.Version` - Git tag (or "dev")
- `main.GitCommit` - Short commit hash
- `main.BuildDate` - UTC build timestamp

**In your Go code:**

```go
package main

var (
    Version   = "dev"
    GitCommit = "unknown"
    BuildDate = "unknown"
)

func main() {
    fmt.Printf("Version: %s (%s) built on %s\n", Version, GitCommit, BuildDate)
}
```

### Incremental Builds

Skip rebuilding platforms that already have binaries:

```bash
./go-multi-build -i
```

Useful for:
- Iterative development
- Adding new platforms to existing builds
- Resuming interrupted builds

### Binary Compression

Compress binaries with UPX to reduce file size:

```bash
./go-multi-build -c
```

**Requirements:**
- UPX must be installed (`brew install upx` or `apt-get install upx`)

**Compression ratios:**
- Typically 40-60% size reduction
- No runtime performance impact (decompresses to memory)

### Build Summary

Automatically displayed after builds complete:

```
Build Summary:
─────────────────────────────────────────────────
Status Platform              Size       Time
─────────────────────────────────────────────────
✓      linux/amd64          8.2M       3.45s
✓      darwin/arm64         8.1M       3.78s
⊙      windows/amd64        8.4M       0.00s
─────────────────────────────────────────────────
Total time: 7.23s
```

Legend:
- ✓ = Successfully built
- ⊙ = Skipped (incremental build)
- ✗ = Failed

### Color Output

Color-coded output for better readability:
- **Green**: Success messages
- **Red**: Error messages
- **Yellow**: Warnings
- **Cyan**: Info messages
- **Blue**: Verbose output

Disable with `--no-color` for:
- CI/CD logs
- Piping to files
- Terminal compatibility issues

---

## Configuration

### Configuration Priority

Settings are applied in this order (later overrides earlier):

1. **Defaults** - Built into the script
2. **Config File** - `go-multi-build.conf`
3. **Environment Variables** - `GO_BUILD_PLATFORMS`, etc.
4. **Command-Line Flags** - Highest priority

### Config File

Create `go-multi-build.conf` in your project root:

```bash
# Platforms to build
PLATFORMS="linux/amd64,linux/arm64,darwin/amd64,darwin/arm64,windows/amd64"

# Output directory
OUTPUT_DIR="./dist"

# Enable parallel builds
PARALLEL=true

# Enable compression
COMPRESS=true

# Custom ldflags
CUSTOM_LDFLAGS='-s -w -extldflags "-static"'
```

See [configuration.md](./configuration.md) for all options.

### Environment Variables

```bash
# Set platforms
export GO_BUILD_PLATFORMS="linux/amd64,windows/amd64"

# Run build
./go-multi-build
```

### Command-Line Flags

Flags always take highest priority:

```bash
./go-multi-build -P -p "linux/amd64,darwin/arm64" -o build
```

See [commands.md](./commands.md) for complete flag reference.

---

## Usage Patterns

### Development Workflow

Quick local builds for testing:

```bash
# Build only for your current platform
./go-multi-build -p "$(go env GOOS)/$(go env GOARCH)"

# Or use config file for dev
# go-multi-build.conf:
PLATFORMS="darwin/arm64"  # Your local platform
OUTPUT_DIR="./build"
```

### CI/CD Integration

Example GitHub Actions workflow:

```yaml
- name: Build binaries
  run: |
    ./go-multi-build -P -q -v -o dist
```

Example GitLab CI:

```yaml
build:
  script:
    - ./go-multi-build -P -q -o dist
  artifacts:
    paths:
      - dist/
```

**Recommended flags for CI/CD:**
- `-q` - Quiet mode (less log spam)
- `-P` - Parallel (faster builds)
- `--no-color` - Clean logs

### Release Builds

Full production release with all features:

```bash
./go-multi-build \
  -P \
  -v \
  -c \
  --cleanup \
  -o release \
  -p "linux/amd64,linux/arm64,darwin/amd64,darwin/arm64,windows/amd64,windows/arm64"
```

Or use a config file:

```bash
# release.conf
PLATFORMS="linux/amd64,linux/arm64,darwin/amd64,darwin/arm64,windows/amd64,windows/arm64"
OUTPUT_DIR="./release"
PARALLEL=true
COMPRESS=true
EMBED_VERSION=true
CUSTOM_LDFLAGS='-s -w -extldflags "-static"'
```

Then:

```bash
CONFIG_FILE=release.conf ./go-multi-build --cleanup
```

---

## Advanced Topics

### Custom Build Flags

Pass custom ldflags to strip symbols and reduce binary size:

```bash
./go-multi-build -l '-s -w -extldflags "-static"'
```

Common ldflags:
- `-s` - Strip symbol table
- `-w` - Strip DWARF debugging info
- `-X main.Var=value` - Set string variable
- `-extldflags "-static"` - Static linking

### Cross-Compilation Caveats

**CGO Limitations:**

`go-multi-builder` sets `CGO_ENABLED=0` for pure Go cross-compilation. If your project requires CGO:

1. You'll need platform-specific toolchains
2. Consider building on native platforms instead
3. Or use Docker with cross-compilation toolchains

**Platform-Specific Issues:**

- **Windows**: Produces `.exe` files automatically
- **Android**: Requires Android NDK
- **iOS**: Cannot cross-compile (requires Xcode)

### Debugging Build Issues

**Verbose Mode:**

```bash
./go-multi-build -V
```

Shows:
- Go version
- Directory operations
- Compression steps
- All build output

**Dry Run:**

Preview what will be built:

```bash
./go-multi-build --dry-run
```

**Common Issues:**

1. **"Go is not installed"**
   - Solution: Install Go from https://golang.org/dl/

2. **"go.mod not found"**
   - Solution: Run `go mod init` in your project

3. **Build fails for specific platform**
   - Check for platform-specific code
   - Use build tags to exclude problematic code

---

## Reference Documentation

- **[Configuration Reference](./configuration.md)** - Complete configuration options
- **[Command Reference](./commands.md)** - All command-line flags and arguments
- **[Platform Support](./platforms.md)** - Supported GOOS/GOARCH combinations

---

## Troubleshooting

### Build Fails Immediately

**Problem**: Script exits with error before building

**Solutions**:
1. Ensure Go is installed: `go version`
2. Check you're in a Go module: `ls go.mod`
3. Run with verbose mode: `./go-multi-build -V`

### Builds Succeed But Binary Doesn't Run

**Problem**: Binary built but won't execute on target platform

**Solutions**:
1. Verify you built for correct platform
2. Check for CGO dependencies (requires native builds)
3. Ensure target has required shared libraries
4. Try static linking: `-l '-extldflags "-static"'`

### Parallel Builds Hang

**Problem**: Using `-P` flag causes builds to hang

**Solutions**:
1. Try without `-P` to isolate issue
2. Check available memory
3. Reduce number of platforms
4. Check for zombie processes: `ps aux | grep go`

### Version Embedding Not Working

**Problem**: `-v` flag doesn't embed version info

**Solutions**:
1. Ensure you're in a git repository
2. Check you have at least one tag: `git tag`
3. Variables must be in `main` package
4. Variables must be declared as `var Version string`

### Colors Not Displaying

**Problem**: Output is plain text without colors

**Solutions**:
1. Check terminal supports colors
2. Remove `--no-color` flag if set
3. Check if output is piped (auto-disables colors)
4. Verify `$TERM` is set correctly

---

For more help, please [open an issue](https://github.com/acockrell/go-multi-builder/issues) on GitHub.
