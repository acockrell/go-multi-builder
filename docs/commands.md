# Command Reference

Complete reference for all `go-multi-builder` command-line options.

## Table of Contents

- [Syntax](#syntax)
- [Arguments](#arguments)
- [Flags](#flags)
  - [Help and Information](#help-and-information)
  - [Platform Selection](#platform-selection)
  - [Output Options](#output-options)
  - [Build Options](#build-options)
  - [Optimization Options](#optimization-options)
  - [Output Control](#output-control)
  - [Utility Options](#utility-options)
- [Flag Combinations](#flag-combinations)
- [Examples](#examples)

---

## Syntax

```bash
go-multi-build [OPTIONS] [package-name]
```

**Components**:
- `OPTIONS` - Zero or more command-line flags
- `package-name` - Optional Go package name (auto-detected from go.mod if omitted)

---

## Arguments

### package-name

**Optional**
**Type**: String
**Description**: Go package to build

```bash
./go-multi-build myapp
./go-multi-build github.com/user/project
```

**Auto-detection**:
If omitted, the package name is read from `go.mod` in the current directory:

```bash
# Reads package from go.mod
./go-multi-build
```

**Format**:
- Simple name: `myapp`
- Full module path: `github.com/user/myapp`
- Local path: `./cmd/myapp`

---

## Flags

### Help and Information

#### `-h`, `--help`

Show help message and exit

```bash
./go-multi-build --help
```

**Output**: Displays usage information, all flags, examples, and configuration precedence

---

### Platform Selection

#### `-p LIST`, `--platforms LIST`

Specify platforms to build for

**Type**: Comma-separated list of `GOOS/GOARCH` pairs
**Default**: `linux/amd64,darwin/amd64,darwin/arm64`

```bash
./go-multi-build -p "linux/amd64,darwin/arm64"
./go-multi-build --platforms "windows/amd64,linux/amd64"
```

**Common platforms**:
- `linux/amd64` - Linux 64-bit (Intel/AMD)
- `linux/arm64` - Linux 64-bit (ARM)
- `darwin/amd64` - macOS Intel
- `darwin/arm64` - macOS Apple Silicon
- `windows/amd64` - Windows 64-bit

See [platforms.md](./platforms.md) for all supported combinations.

---

### Output Options

#### `-o DIR`, `--output-dir DIR`

Specify output directory for binaries

**Type**: Directory path
**Default**: `.` (current directory)

```bash
./go-multi-build -o build
./go-multi-build --output-dir ./dist
./go-multi-build -o /tmp/builds
```

**Behavior**:
- Directory is created if it doesn't exist
- Relative paths are relative to current working directory
- Binaries are named: `<package>-<GOOS>-<GOARCH>[.exe]`

---

### Build Options

#### `-P`, `--parallel`

Build all platforms in parallel

**Type**: Boolean flag (no argument)
**Default**: Sequential builds

```bash
./go-multi-build -P
./go-multi-build --parallel
```

**Effect**:
- Launches all platform builds simultaneously
- Speeds up builds on multi-core systems
- Uses more memory
- Output may be interleaved

**Recommended when**:
- Building 3+ platforms
- Multi-core CPU available
- Sufficient RAM (2GB+ per platform)

#### `-l FLAGS`, `--ldflags FLAGS`

Custom ldflags for go build

**Type**: String (quoted if contains spaces)
**Default**: `'-extldflags "-static"'`

```bash
./go-multi-build -l '-s -w'
./go-multi-build --ldflags '-X main.Version=1.0.0'
./go-multi-build -l '-s -w -extldflags "-static"'
```

**Common flags**:
- `-s` - Strip symbol table
- `-w` - Strip DWARF debugging information
- `-X main.Var=value` - Set string variable
- `-extldflags "-static"` - Produce statically-linked binary

**Note**: Version flags are prepended automatically when using `-v`

#### `-s`, `--strip`

Strip debug symbols from binaries

**Type**: Boolean flag (no argument)
**Default**: Debug symbols included

```bash
./go-multi-build -s
./go-multi-build --strip
```

**Effect**:
- Adds `-s -w` to ldflags automatically
- `-s` strips symbol table
- `-w` strips DWARF debugging information
- Reduces binary size by 30-50%
- Cannot use debugger on stripped binaries

**Recommended for**:
- Production releases
- When binary size matters
- Distribution to end users

**Not recommended for**:
- Development builds
- When debugging is needed
- Troubleshooting crashes

---

### Optimization Options

#### `-v`, `--version`

Embed git version information into binaries

**Type**: Boolean flag (no argument)
**Default**: Disabled

```bash
./go-multi-build -v
./go-multi-build --version
```

**Requirements**:
- Must be in a git repository
- Requires these variables in your `main` package:
  ```go
  var (
      Version   = "dev"
      GitCommit = "unknown"
      BuildDate = "unknown"
  )
  ```

**What gets embedded**:
- `main.Version` - Git tag (or "dev")
- `main.GitCommit` - Short commit hash
- `main.BuildDate` - Build timestamp (UTC)

#### `-i`, `--incremental`

Skip platforms that already have binaries

**Type**: Boolean flag (no argument)
**Default**: Rebuild all platforms

```bash
./go-multi-build -i
./go-multi-build --incremental
```

**Behavior**:
- Checks if binary file exists
- Skips build if found
- Shows skipped platforms in summary

**Use cases**:
- Adding new platforms to existing builds
- Resuming interrupted builds
- Iterative development

#### `-c`, `--compress`

Compress binaries with UPX

**Type**: Boolean flag (no argument)
**Default**: No compression

```bash
./go-multi-build -c
./go-multi-build --compress
```

**Requirements**:
- UPX must be installed
- Install: `brew install upx` (macOS) or `apt-get install upx` (Linux)

**Effect**:
- Typically 40-60% size reduction
- Slower build times
- No runtime performance penalty

---

### Output Control

#### `-V`, `--verbose`

Enable verbose output

**Type**: Boolean flag (no argument)
**Default**: Normal output

```bash
./go-multi-build -V
./go-multi-build --verbose
```

**Shows**:
- Go version being used
- Directory operations
- Complete build output
- Compression progress
- File removal during cleanup

**Use when**:
- Debugging build issues
- Understanding what the script is doing
- Troubleshooting

#### `-q`, `--quiet`

Minimal output (errors only)

**Type**: Boolean flag (no argument)
**Default**: Normal output

```bash
./go-multi-build -q
./go-multi-build --quiet
```

**Shows only**:
- Error messages
- Warning messages (to stderr)

**Suppresses**:
- Info messages
- Success messages
- Build summary table

**Use when**:
- Running in CI/CD
- Scripting/automation
- Piping output

**Note**: Cannot combine with `-V` (verbose). Quiet takes precedence.

#### `--no-color`

Disable colored output

**Type**: Boolean flag (no argument)
**Default**: Colors enabled (if TTY)

```bash
./go-multi-build --no-color
```

**Use when**:
- CI/CD doesn't support ANSI codes
- Logging to files
- Terminal incompatibility

**Note**: Colors are automatically disabled when output is not a TTY

---

### Artifact Generation

#### `--checksums`

Generate SHA256 checksum files for each binary

**Type**: Boolean flag (no argument)
**Default**: No checksums generated

```bash
./go-multi-build --checksums
```

**Output**:
- Creates `.sha256` file for each binary
- Format: `<checksum>  <filename>`
- Example: `myapp-linux-amd64.sha256`

**Use cases**:
- Verify download integrity
- Detect file corruption
- Security compliance
- Release verification

#### `--archive`

Create release archives for binaries

**Type**: Boolean flag (no argument)
**Default**: No archives created

```bash
./go-multi-build --archive
```

**Output**:
- `.tar.gz` for Unix platforms (Linux, macOS, BSD)
- `.zip` for Windows platforms
- Archives contain the binary

**Examples**:
- `myapp-linux-amd64.tar.gz`
- `myapp-windows-amd64.exe.zip`

**Use cases**:
- Distribution packages
- Release artifacts
- Easy downloads
- Organized deployments

#### `--manifest`

Generate build manifest JSON file

**Type**: Boolean flag (no argument)
**Default**: No manifest generated

```bash
./go-multi-build --manifest
```

**Output**:
- Creates `build-manifest.json` in output directory
- Contains complete build metadata

**Manifest includes**:
- Build date and total time
- Package name and git information
- Array of all builds with:
  - Platform (GOOS/GOARCH)
  - Status (OK/SKIP/FAIL)
  - Binary filename
  - File size (human and bytes)
  - Build time
  - SHA256 checksum (if --checksums used)

**Example manifest**:
```json
{
  "build_date": "2025-11-15T12:34:56Z",
  "build_time": "7.23s",
  "package": "github.com/user/myapp",
  "package_name": "myapp",
  "git_tag": "v1.2.3",
  "git_commit": "a1b2c3d...",
  "output_dir": "./build",
  "builds": [
    {
      "platform": "linux/amd64",
      "goos": "linux",
      "goarch": "amd64",
      "status": "OK",
      "binary": "myapp-linux-amd64",
      "size": "8.2M",
      "size_bytes": 8601234,
      "build_time": "3.45s",
      "sha256": "a1b2c3d..."
    }
  ]
}
```

**Use cases**:
- Build automation tracking
- Release documentation
- CI/CD integration
- Audit trails

---

### Utility Options

#### `-d`, `--dry-run`

Preview what would be built without building

**Type**: Boolean flag (no argument)
**Default**: Actually build

```bash
./go-multi-build --dry-run
./go-multi-build -d
```

**Shows**:
- Package name
- Output directory
- All configuration settings
- List of platforms that would be built
- Output file names

**Use when**:
- Testing configuration
- Validating platform list
- Checking output paths

#### `--cleanup`

Remove old binaries before building

**Type**: Boolean flag (no argument)
**Default**: Keep existing files

```bash
./go-multi-build --cleanup
```

**Behavior**:
- Removes all matching binary files before building
- Only removes files for platforms being built
- Shows removed files in verbose mode

**Use when**:
- Ensuring clean builds
- Removing stale binaries
- Starting fresh

---

## Flag Combinations

### Common Combinations

#### Development Build
```bash
./go-multi-build -i -V
```
- Incremental (skip existing)
- Verbose (see what's happening)

#### Production Release
```bash
./go-multi-build -P -v -s -c --checksums --archive --manifest --cleanup -o release
```
- Parallel (fast)
- Version embedded
- Stripped symbols (small)
- Compressed (smaller)
- Checksums generated
- Archives created
- Manifest file
- Clean build
- Output to `release/`

#### CI/CD Build
```bash
./go-multi-build -P -q --no-color -o dist
```
- Parallel (fast)
- Quiet (minimal logs)
- No colors (clean logs)
- Output to `dist/`

#### Quick Test Build
```bash
./go-multi-build -p "$(go env GOOS)/$(go env GOARCH)" -V
```
- Current platform only
- Verbose output

#### Multi-Platform Release
```bash
./go-multi-build -P -v -c -o build \
  -p "linux/amd64,linux/arm64,darwin/amd64,darwin/arm64,windows/amd64"
```
- All major platforms
- Parallel build
- Version embedded
- Compressed

---

## Examples

### Basic Examples

**Simplest usage**:
```bash
./go-multi-build
```

**Specify package**:
```bash
./go-multi-build myapp
```

**Build specific platforms**:
```bash
./go-multi-build -p "linux/amd64,windows/amd64"
```

**Output to directory**:
```bash
./go-multi-build -o build
```

### Intermediate Examples

**Parallel build with version**:
```bash
./go-multi-build -P -v
```

**Incremental build to custom directory**:
```bash
./go-multi-build -i -o dist
```

**Compressed builds**:
```bash
./go-multi-build -c -P
```

**Quiet mode for scripts**:
```bash
./go-multi-build -q -P -o /tmp/builds
```

### Advanced Examples

**Full production release**:
```bash
./go-multi-build \
  --platforms "linux/amd64,linux/arm64,darwin/amd64,darwin/arm64,windows/amd64,windows/arm64" \
  --parallel \
  --version \
  --compress \
  --cleanup \
  --output-dir ./release \
  --ldflags '-s -w -extldflags "-static"'
```

**Custom ldflags with version**:
```bash
./go-multi-build -v -l '-X main.AppName=MyApp -X main.License=MIT -s -w'
```

**Dry run with verbose**:
```bash
./go-multi-build -d -V -p "linux/amd64,darwin/arm64,windows/amd64"
```

**CI/CD optimized**:
```bash
./go-multi-build \
  --quiet \
  --parallel \
  --no-color \
  --output-dir ./artifacts \
  --platforms "linux/amd64,darwin/amd64,windows/amd64"
```

**Single platform verbose debug**:
```bash
./go-multi-build -V -p "$(go env GOOS)/$(go env GOARCH)" -o /tmp/test
```

### Using with Environment Variables

**Override platforms via env**:
```bash
GO_BUILD_PLATFORMS="linux/amd64,windows/amd64" ./go-multi-build -P
```

**Combine env and flags**:
```bash
export GO_BUILD_PLATFORMS="linux/amd64,darwin/arm64"
./go-multi-build -P -v -o build
```

**Custom config file**:
```bash
CONFIG_FILE=release.conf ./go-multi-build --cleanup
```

---

## Exit Codes

- `0` - Success (all builds completed)
- `1` - Error (validation failed, build failed, etc.)

---

For configuration file options, see [configuration.md](./configuration.md).
For platform combinations, see [platforms.md](./platforms.md).
