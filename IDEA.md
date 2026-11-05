## Project description

A CI-driven pipeline that builds and distributes static [aria2](https://aria2.github.io) (`aria2c`) binaries for eight platforms (Linux, macOS, Windows, FreeBSD x amd64/arm64). A pre-baked Ubuntu build image (`ghcr.io/binmgr/aria2:build`) holds all cross-compile toolchains and statically built dependencies; binaries are then built from the official upstream release tarball. Published artifacts are standalone binaries, a SHA-256 `checksums.txt`, and a scratch-based Docker runtime image.

## Project variables

project_name: aria2
project_org: binmgr
internal_name: aria2
internal_org: binmgr
upstream_source: https://github.com/aria2/aria2
registry: ghcr.io
image_name: binmgr/aria2
supported_platforms: linux-amd64, linux-arm64, darwin-amd64, darwin-arm64, windows-amd64, windows-arm64, freebsd-amd64, freebsd-arm64

## Business logic

**Must have:**
- Static `aria2c` for all eight targets (macOS links system libc/libSystem only)
- Features compiled in: HTTPS (OpenSSL), zlib, async DNS (c-ares), SFTP (libssh2), XML-RPC/Metalink (expat), cookies (sqlite3), BitTorrent
- `checksums.txt` (SHA-256) covering every release asset
- Docker runtime image `ghcr.io/binmgr/aria2:latest` for linux/amd64 and linux/arm64
- Builds on a monthly schedule, on build-image changes, and manually with an optional version override

**Must NOT have:**
- Patched aria2 source — upstream release tarballs only
- Dynamically linked dependencies on Linux, Windows, and FreeBSD

**Constraints:**
- OpenSSL stays on the 3.x LTS line until aria2 supports 4.x
- Dependency versions are pinned in `docker/Dockerfile.build`; change them deliberately
