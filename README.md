# go-multi-builder

This script builds a golang project and is capable of producing binaries for multiple
platforms in a single run.  


# GOOS and GOARCH options
The following table shows the possible combinations of **GOOS** and **GOARCH** you can use:

| GOOS - Target Operating System  | GOARCH - Target Platform | 
| ------------------------------- | ------------------------ |
| android | arm |
| darwin | 386 |
| darwin | amd64 |
| darwin | arm |
| darwin | arm64 |
| dragonfly | amd64 |
| freebsd | 386 |
| freebsd | amd64 |
| freebsd | arm |
| linux | 386 |
| linux | amd64 |
| linux | arm |
| linux | arm64 |
| linux | ppc64 |
| linux | ppc64le |
| linux | mips |
| linux | mipsle |
| linux | mips64 |
| linux | mips64le |
| netbsd | 386 |
| netbsd | amd64 |
| netbsd | arm |
| openbsd | 386 |
| openbsd | amd64 |
| openbsd | arm |
| plan9 | 386 |
| plan9 | amd64 |
| solaris | amd64 |
| windows | 386 |
| windows | amd64 |

**Warning**: Cross-compiling executables for Android requires the Android NDK, and some additional setup which is beyond the scope of this tutorial.