# TODO - go-multi-builder Improvements

## Critical Bugs

- [x] Fix `CGO_ENABLED` and `GOCACHE` - not exported so they have no effect (lines 40-41)
- [x] Remove or define undefined `${WORKSPACE}` variable (line 41)
- [x] Fix typo: "reamote" → "remote" (line 19)
- [x] Fix output message: "for X on Y" → "for X/Y" (line 42)

## Core Features

### Configuration
- [x] Make platforms configurable via `go-multi-build.conf` or env var
- [x] Support custom ldflags from config or CLI
- [x] Add command-line flags: `--platforms`, `--output-dir`, `--parallel`, `--compress`
- [x] Add `-h/--help` flag
- [x] Add dry-run mode

### Build Improvements
- [x] Implement parallel builds for faster compilation
- [x] Organize output into directories (e.g., `build/linux-amd64/`)
- [x] Embed version information using git tags/commits
- [x] Add incremental builds (skip already-built platforms)
- [x] Add build time tracking and display

### Output Artifacts
- [ ] Generate SHA256 checksums for each binary
- [ ] Create release archives (`.tar.gz`, `.zip`)
- [ ] Generate build manifest file
- [ ] Add option to strip debug symbols (`-ldflags "-s -w"`)
- [x] Optional UPX compression

### User Experience
- [x] Auto-detect package name from `go.mod` if not provided
- [x] Validate Go installation and project directory
- [x] Add color output for success/failure
- [x] Show build summary table (platforms, sizes, times)
- [x] Add verbose/quiet modes
- [x] Add cleanup option for old builds

## Code Quality

- [ ] Simplify package name parsing (use `basename` instead of array logic)
- [ ] Add `set -o pipefail` for better error detection
- [ ] Consistent quoting in conditionals
- [ ] Consistent variable bracing

## Advanced Features (Future)

- [ ] Cross-compilation validation
- [ ] Integration with goreleaser-style configuration
- [ ] Custom binary naming patterns
- [ ] Verify binaries are executable after building
