# go-multi-builder

A powerful Bash script for building Go projects across multiple platforms in a single run. Build for Linux, macOS, Windows, and more with parallel execution, version embedding, and comprehensive build reporting.

## Features

- 🚀 **Parallel Builds** - Build multiple platforms simultaneously
- 📦 **Version Embedding** - Automatically embed git version info
- 🎨 **Colored Output** - Beautiful, readable build status
- 📊 **Build Summary** - Detailed table of all builds
- ⚙️ **Highly Configurable** - CLI flags, env vars, or config file
- 🔄 **Incremental Builds** - Skip already-built platforms
- 🗜️ **Compression** - Optional UPX binary compression
- ✅ **Validation** - Checks for Go installation and project structure

## Quick Start

```bash
# Basic usage (auto-detects package from go.mod)
./go-multi-build

# Build specific platforms in parallel
./go-multi-build -P -p "linux/amd64,darwin/arm64,windows/amd64"

# Full-featured build
./go-multi-build -P -v -i -o build myapp

# See all options
./go-multi-build --help
```

## Installation

1. Download the script:
   ```bash
   curl -O https://raw.githubusercontent.com/acockrell/go-multi-builder/main/go-multi-build
   chmod +x go-multi-build
   ```

2. (Optional) Copy the sample config:
   ```bash
   curl -O https://raw.githubusercontent.com/acockrell/go-multi-builder/main/go-multi-build.conf.example
   cp go-multi-build.conf.example go-multi-build.conf
   # Edit go-multi-build.conf to customize
   ```

## Common Usage Examples

```bash
# Auto-detect package and build with defaults
./go-multi-build

# Parallel build to custom directory
./go-multi-build -P -o build

# Embed version info from git
./go-multi-build -v myapp

# Incremental build (skip existing)
./go-multi-build -i

# Clean rebuild with compression
./go-multi-build --cleanup -c

# Quiet mode for CI/CD
./go-multi-build -q -P

# Verbose output for debugging
./go-multi-build -V

# Dry run to preview
./go-multi-build --dry-run
```

## Command-Line Options

| Flag | Description |
|------|-------------|
| `-h, --help` | Show help message |
| `-p, --platforms LIST` | Comma-separated platforms (e.g., "linux/amd64,darwin/arm64") |
| `-o, --output-dir DIR` | Output directory for binaries |
| `-P, --parallel` | Build platforms in parallel |
| `-v, --version` | Embed git version info (tag, commit, date) |
| `-i, --incremental` | Skip already-built platforms |
| `-c, --compress` | Compress binaries with UPX |
| `-s, --strip` | Strip debug symbols (smaller binaries) |
| `-l, --ldflags FLAGS` | Custom ldflags for go build |
| `-V, --verbose` | Verbose output |
| `-q, --quiet` | Quiet mode (minimal output) |
| `-d, --dry-run` | Preview what would be built |
| `--cleanup` | Remove old builds before building |
| `--checksums` | Generate SHA256 checksum files |
| `--archive` | Create release archives (.tar.gz/.zip) |
| `--manifest` | Generate build manifest JSON file |
| `--no-color` | Disable colored output |

## Configuration

Platforms can be configured via (in order of precedence):

1. `--platforms` flag
2. `GO_BUILD_PLATFORMS` environment variable
3. `go-multi-build.conf` configuration file
4. Default: `linux/amd64`, `darwin/amd64`, `darwin/arm64`

### Example Config File

```bash
# go-multi-build.conf
PLATFORMS="linux/amd64,linux/arm64,darwin/amd64,darwin/arm64,windows/amd64"
OUTPUT_DIR="./dist"
PARALLEL=true
COMPRESS=true
```

## Documentation

For more detailed documentation, see the [docs](./docs) directory:

- [Complete Guide](./docs/README.md) - Comprehensive documentation with TOC
- [Configuration Reference](./docs/configuration.md) - All configuration options
- [Command Reference](./docs/commands.md) - Detailed command syntax
- [Platform Support](./docs/platforms.md) - Supported OS/Architecture combinations

## Requirements

- Bash 4.0+
- Go 1.11+ (for Go modules)
- Optional: `upx` for binary compression
- Optional: `git` for version embedding

## Example Output

```
Auto-detected package from go.mod: github.com/user/myapp
Version info: v1.2.3 (a1b2c3d) built on 2025-11-15T12:34:56Z
Building platforms in parallel...
Building github.com/user/myapp for linux/amd64...
Building github.com/user/myapp for darwin/arm64...
Building github.com/user/myapp for windows/amd64...
✓ Built ./build/myapp-linux-amd64 (8.2M, 3.45s)
✓ Built ./build/myapp-darwin-arm64 (8.1M, 3.78s)
✓ Built ./build/myapp-windows-amd64.exe (8.4M, 4.12s)

✓ All builds completed successfully in 4.15s

Build Summary:
─────────────────────────────────────────────────
Status Platform              Size       Time
─────────────────────────────────────────────────
✓      linux/amd64          8.2M       3.45s
✓      darwin/arm64         8.1M       3.78s
✓      windows/amd64        8.4M       4.12s
─────────────────────────────────────────────────
Total time: 4.15s
```

## License

MIT License - See LICENSE file for details

## Contributing

Contributions welcome! Please feel free to submit a Pull Request.
