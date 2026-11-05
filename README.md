# aria2 Static Builds

Static, stripped [aria2](https://aria2.github.io) (`aria2c`) binaries for Linux, Windows, macOS, and FreeBSD (amd64/arm64), built from upstream release tarballs and published automatically.

| Platform | amd64 | arm64 |
|----------|-------|-------|
| Linux | `aria2c-linux-amd64` | `aria2c-linux-arm64` |
| Windows | `aria2c-windows-amd64.exe` | `aria2c-windows-arm64.exe` |
| macOS | `aria2c-darwin-amd64` | `aria2c-darwin-arm64` |
| FreeBSD | `aria2c-freebsd-amd64` | `aria2c-freebsd-arm64` |

Download from [Releases](../../releases/latest); verify with `sha256sum -c checksums.txt`.

## Compiled-in features

OpenSSL (HTTPS/TLS), zlib, c-ares (async DNS), libssh2 (SFTP), expat (XML-RPC, Metalink), SQLite3 (Firefox/Chrome cookies), BitTorrent.

## Docker

```bash
docker run --rm -v "$PWD:/downloads" ghcr.io/binmgr/aria2:latest https://example.com/file.iso
```

The image is `FROM scratch` plus CA certificates (linux/amd64, linux/arm64).

## Build locally

```bash
make build-env
make build TARGET=linux-amd64 VERSION=1.37.0
```

Output lands in `binaries/`. Targets: `{linux,windows,darwin,freebsd}-{amd64,arm64}`.

## CI

- `build-env-image.yml` builds `ghcr.io/binmgr/aria2:build` (toolchains and static deps) on `docker/Dockerfile.build` changes and quarterly.
- `build-binaries.yml` builds all eight targets, publishes the release and runtime image monthly, after a build-image update, or manually.

## License

Build scripts: MIT. aria2: GPL-2.0-or-later. See [LICENSE.md](LICENSE.md).
