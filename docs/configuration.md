# Configuration Reference

Complete reference for all `go-multi-builder` configuration options.

## Table of Contents

- [Configuration Methods](#configuration-methods)
- [Configuration Priority](#configuration-priority)
- [Config File Options](#config-file-options)
- [Environment Variables](#environment-variables)
- [Example Configurations](#example-configurations)

---

## Configuration Methods

`go-multi-builder` can be configured in three ways:

1. **Config File** - `go-multi-build.conf` in project directory
2. **Environment Variables** - Exported in your shell
3. **Command-Line Flags** - Passed when running the script

**Quick Setup**: Use `--save-config` to save command-line options to the config file:
```bash
./go-multi-build -P -p "linux/amd64,darwin/arm64" -o dist --save-config
```
This creates or updates `go-multi-build.conf` with your settings. See [commands.md](./commands.md#--save-config) for details.

## Configuration Priority

When the same setting is specified multiple ways, this order determines which value is used (highest priority first):

1. **Command-Line Flags** - Always wins
2. **Environment Variables** - Overrides config file
3. **Config File** - Overrides defaults
4. **Built-in Defaults** - Fallback values

### Example

```bash
# Config file says:
PLATFORMS="linux/amd64"

# Environment variable says:
export GO_BUILD_PLATFORMS="darwin/arm64"

# Command-line flag says:
./go-multi-build -p "windows/amd64"

# Result: Builds for windows/amd64 (CLI flag wins)
```

---

## Config File Options

Create a file named `go-multi-build.conf` in your project root or specify a custom location:

```bash
CONFIG_FILE=my-config.conf ./go-multi-build
```

### PLATFORMS

**Type**: String (comma-separated list)
**Default**: `"linux/amd64,darwin/amd64,darwin/arm64"`
**Description**: List of platforms to build for

```bash
PLATFORMS="linux/amd64,linux/arm64,darwin/amd64,darwin/arm64,windows/amd64"
```

**Notes**:
- Format is `GOOS/GOARCH` pairs
- No spaces around commas
- See [platforms.md](./platforms.md) for all supported combinations

### OUTPUT_DIR

**Type**: String (directory path)
**Default**: `"."` (current directory)
**Description**: Directory where binaries will be created

```bash
OUTPUT_DIR="./build"
OUTPUT_DIR="./dist"
OUTPUT_DIR="/tmp/builds"
```

**Notes**:
- Directory is created if it doesn't exist
- Relative paths are relative to where script is run
- Use absolute paths for consistency

### BINARY_NAME_PATTERN

**Type**: String (pattern with variables)
**Default**: `"{name}-{os}-{arch}"`
**Description**: Custom naming pattern for output binaries

```bash
BINARY_NAME_PATTERN="{name}-{version}-{os}-{arch}"
BINARY_NAME_PATTERN="{name}_{date}_{os}_{arch}"
BINARY_NAME_PATTERN="{os}/{arch}/{name}"
```

**Available Variables**:
- `{name}` or `{package}` - Package/binary name
- `{os}` or `{goos}` - Operating system (linux, darwin, windows, etc.)
- `{arch}` or `{goarch}` - Architecture (amd64, arm64, 386, arm, etc.)
- `{version}` - Git tag/version (requires git repository)
- `{commit}` - Git commit hash (short, requires git repository)
- `{date}` - Build date in YYYYMMDD format
- `{time}` - Build time in HHMMSS format (UTC)

**Examples**:

```bash
# Version in filename
BINARY_NAME_PATTERN="{name}-{version}-{os}-{arch}"
# Produces: myapp-v1.2.3-linux-amd64

# Date-based builds
BINARY_NAME_PATTERN="{name}_{date}_{os}_{arch}"
# Produces: myapp_20251115_linux_amd64

# Organized by platform (creates subdirectories)
BINARY_NAME_PATTERN="{os}/{arch}/{name}"
# Produces: linux/amd64/myapp, darwin/arm64/myapp

# Release naming with commit hash
BINARY_NAME_PATTERN="{name}-{version}-{os}-{arch}-{commit}"
# Produces: myapp-v1.2.3-linux-amd64-a1b2c3d
```

**Notes**:
- `.exe` extension automatically added for Windows
- Directory separators (`/`) in pattern create subdirectories
- Pattern is validated at startup
- Warning if `{os}` or `{arch}` not included (builds may overwrite)
- Invalid characters (`: * ? " < > |`) are rejected

**Use cases**:
- Version-tagged releases
- Date-stamped nightly builds
- Organized directory structures
- CI/CD build numbering
- Custom naming conventions

### PARALLEL

**Type**: Boolean (`true` or `false`)
**Default**: `false`
**Description**: Build all platforms simultaneously

```bash
PARALLEL=true
```

**Advantages**:
- Faster builds on multi-core systems
- Better resource utilization

**Disadvantages**:
- Harder to debug build failures
- Higher memory usage
- Interleaved output in verbose mode

### COMPRESS

**Type**: Boolean (`true` or `false`)
**Default**: `false`
**Description**: Compress binaries with UPX after building

```bash
COMPRESS=true
```

**Requirements**:
- UPX must be installed
- Install: `brew install upx` or `apt-get install upx`

**Effect**:
- Typically 40-60% size reduction
- Slightly slower builds
- No runtime performance impact

### INCREMENTAL

**Type**: Boolean (`true` or `false`)
**Default**: `false`
**Description**: Skip platforms that already have binaries built

```bash
INCREMENTAL=true
```

**Use cases**:
- Adding new platforms to existing build
- Iterative development
- Resuming interrupted builds

### EMBED_VERSION

**Type**: Boolean (`true` or `false`)
**Default**: `false`
**Description**: Embed git version information into binaries

```bash
EMBED_VERSION=true
```

**Requirements**:
- Must be in a git repository
- `main.Version`, `main.GitCommit`, `main.BuildDate` variables in your code

**What it embeds**:
- Git tag (or "dev" if no tags)
- Short commit hash
- Build timestamp in UTC

### CUSTOM_LDFLAGS

**Type**: String (Go linker flags)
**Default**: `'-extldflags "-static"'`
**Description**: Custom ldflags passed to `go build`

```bash
CUSTOM_LDFLAGS='-s -w -extldflags "-static"'
```

**Common flags**:
- `-s` - Strip symbol table (smaller binary)
- `-w` - Strip DWARF debug info (smaller binary)
- `-X main.Var=value` - Set string variable at compile time
- `-extldflags "-static"` - Static linking

**Note**: If `EMBED_VERSION=true`, version flags are prepended automatically

### STRIP

**Type**: Boolean (`true` or `false`)
**Default**: `false`
**Description**: Strip debug symbols from binaries

```bash
STRIP=true
```

**Effect**:
- Automatically adds `-s -w` to ldflags
- Reduces binary size by 30-50%
- Production-ready binaries

### CHECKSUMS

**Type**: Boolean (`true` or `false`)
**Default**: `false`
**Description**: Generate SHA256 checksum files

```bash
CHECKSUMS=true
```

**Output**:
- Creates `.sha256` file for each binary
- Format: `<checksum>  <filename>`

**Use cases**:
- Verify downloads
- Security compliance
- Release integrity

### ARCHIVE

**Type**: Boolean (`true` or `false`)
**Default**: `false`
**Description**: Create release archives

```bash
ARCHIVE=true
```

**Output**:
- `.tar.gz` for Unix (Linux, macOS, BSD)
- `.zip` for Windows

**Use cases**:
- Distribution packages
- Release artifacts

### MANIFEST

**Type**: Boolean (`true` or `false`)
**Default**: `false`
**Description**: Generate build manifest JSON

```bash
MANIFEST=true
```

**Output**:
- Creates `build-manifest.json`
- Includes all build metadata
- Useful for CI/CD tracking

### VALIDATE

**Type**: Boolean (`true` or `false`)
**Default**: `false`
**Description**: Validate cross-compiled binaries

```bash
VALIDATE=true
```

**Checks**:
- File format (ELF/Mach-O/PE)
- Architecture matches target
- Executable permissions
- Size sanity checks

**Use cases**:
- Ensure build correctness
- Detect cross-compilation issues
- Quality assurance

### VALIDATE_STRICT

**Type**: Boolean (`true` or `false`)
**Default**: `false`
**Description**: Fail build on validation errors

```bash
VALIDATE_STRICT=true
```

**Effect**:
- Enables VALIDATE automatically
- Stops build on validation failure
- Non-zero exit code

**Use cases**:
- CI/CD pipelines
- Release builds
- Automated deployments

### VERBOSE

**Type**: Boolean (`true` or `false`)
**Default**: `false`
**Description**: Enable verbose output

```bash
VERBOSE=true
```

**Shows**:
- Go version
- Directory creation
- File removal during cleanup
- Complete build output
- Compression details

### QUIET

**Type**: Boolean (`true` or `false`)
**Default**: `false`
**Description**: Minimal output (errors only)

```bash
QUIET=true
```

**Use cases**:
- CI/CD pipelines
- Scripting/automation
- When output is logged elsewhere

**Note**: Cannot be used with `VERBOSE` (QUIET takes precedence)

### CLEANUP

**Type**: Boolean (`true` or `false`)
**Default**: `false`
**Description**: Remove old binaries before building

```bash
CLEANUP=true
```

**Use cases**:
- Clean release builds
- Ensuring no stale binaries
- Freeing disk space

### NO_COLOR

**Type**: Boolean (`true` or `false`)
**Default**: `false`
**Description**: Disable colored output

```bash
NO_COLOR=true
```

**When to use**:
- CI/CD systems that don't handle ANSI codes
- Logging to files
- Terminal compatibility issues

**Note**: Colors are automatically disabled if output is not a TTY

---

## Environment Variables

All config file options can be set as environment variables.

### GO_BUILD_PLATFORMS

Equivalent to `PLATFORMS` in config file:

```bash
export GO_BUILD_PLATFORMS="linux/amd64,darwin/arm64,windows/amd64"
./go-multi-build
```

### Other Variables

Most config options work as environment variables:

```bash
export OUTPUT_DIR="./build"
export PARALLEL=true
export COMPRESS=true
export VERBOSE=true

./go-multi-build
```

### CONFIG_FILE

Specify alternate config file location:

```bash
CONFIG_FILE=~/my-builds/config.conf ./go-multi-build
```

---

## Example Configurations

### Development Configuration

Fast local builds for testing:

```bash
# dev.conf
PLATFORMS="$(go env GOOS)/$(go env GOARCH)"
OUTPUT_DIR="./build"
PARALLEL=false
COMPRESS=false
INCREMENTAL=true
VERBOSE=true
```

Usage:
```bash
CONFIG_FILE=dev.conf ./go-multi-build
```

### Production Release Configuration

Optimized binaries for all major platforms with full artifacts and validation:

```bash
# release.conf
PLATFORMS="linux/386,linux/amd64,linux/arm,linux/arm64,darwin/amd64,darwin/arm64,windows/386,windows/amd64,windows/arm64"
OUTPUT_DIR="./release"
PARALLEL=true
COMPRESS=true
STRIP=true
EMBED_VERSION=true
CHECKSUMS=true
ARCHIVE=true
MANIFEST=true
VALIDATE=true
CLEANUP=true
```

Usage:
```bash
CONFIG_FILE=release.conf ./go-multi-build
```

This creates:
- Stripped, compressed binaries
- SHA256 checksum files
- Release archives (.tar.gz/.zip)
- Build manifest JSON with validation results
- Validated cross-compiled binaries

### CI/CD Configuration

Quiet mode for clean logs:

```bash
# ci.conf
PLATFORMS="linux/amd64,darwin/amd64,windows/amd64"
OUTPUT_DIR="./dist"
PARALLEL=true
QUIET=true
NO_COLOR=true
EMBED_VERSION=true
```

Usage:
```bash
CONFIG_FILE=ci.conf ./go-multi-build
```

### Server-Only Configuration

Linux builds for servers:

```bash
# server.conf
PLATFORMS="linux/amd64,linux/arm64"
OUTPUT_DIR="./server-builds"
PARALLEL=true
COMPRESS=true
CUSTOM_LDFLAGS='-s -w -extldflags "-static"'
```

### Desktop-Only Configuration

Builds for desktop platforms:

```bash
# desktop.conf
PLATFORMS="darwin/amd64,darwin/arm64,windows/amd64,linux/amd64"
OUTPUT_DIR="./desktop-builds"
PARALLEL=true
COMPRESS=true
EMBED_VERSION=true
```

### Minimal Size Configuration

Smallest possible binaries:

```bash
# minimal.conf
PLATFORMS="linux/amd64"
OUTPUT_DIR="./minimal"
COMPRESS=true
CUSTOM_LDFLAGS='-s -w -extldflags "-static"'
```

### Testing Configuration

Quick builds without extras:

```bash
# test.conf
PLATFORMS="linux/amd64,darwin/arm64"
OUTPUT_DIR="./test-builds"
PARALLEL=false
INCREMENTAL=true
VERBOSE=true
```

---

## Config File Best Practices

### 1. Use Version Control

Commit your config files to git:

```bash
git add go-multi-build.conf
git commit -m "Add build configuration"
```

### 2. Multiple Configs for Different Use Cases

```bash
# Different configs for different scenarios
dev.conf       # Fast local builds
release.conf   # Production releases
ci.conf        # CI/CD pipelines
```

### 3. Document Your Config

```bash
# go-multi-build.conf

# Production release configuration
# Updated: 2025-11-15
# Builds for all major platforms with optimizations

PLATFORMS="linux/amd64,darwin/amd64,darwin/arm64,windows/amd64"
OUTPUT_DIR="./release"
PARALLEL=true
COMPRESS=true
```

### 4. Use Comments Liberally

```bash
# Server platforms only (no desktop)
PLATFORMS="linux/amd64,linux/arm64"

# Output to release directory
OUTPUT_DIR="./release"

# Enable parallel for speed (we have 8 cores)
PARALLEL=true

# Skip compression (our binaries are already small)
COMPRESS=false
```

### 5. Keep Platform-Specific Configs

```bash
# linux-only.conf
PLATFORMS="linux/386,linux/amd64,linux/arm,linux/arm64"

# darwin-only.conf
PLATFORMS="darwin/amd64,darwin/arm64"

# windows-only.conf
PLATFORMS="windows/386,windows/amd64,windows/arm64"
```

---

For command-line flag reference, see [commands.md](./commands.md).
For platform combinations, see [platforms.md](./platforms.md).
