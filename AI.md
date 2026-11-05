# aria2 — Implementation Spec (HOW)

## Source of truth

Project-specific values live in `IDEA.md`. Placeholders like `{project_name}` resolve from `IDEA.md § Project variables`. Rule overrides live in `SPEC.md`.

---

## Directory layout

```
{project_name}/
├── docker/
│   ├── Dockerfile.build      # Build environment: toolchains + static deps + build-aria2
│   └── Dockerfile            # Runtime image: scratch + CA certs + aria2c (linux amd64/arm64)
├── .github/
│   ├── CODEOWNERS
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   ├── pull_request_template.md
│   ├── SECURITY.md
│   └── workflows/
│       ├── build-env-image.yml   # Builds ghcr.io/{project_org}/{project_name}:build
│       ├── build-binaries.yml    # Builds 8 targets, release, runtime image
│       └── security.yml          # secret-scan + workflow-policy + image-scan
├── .gitea/workflows/             # Gitea mirror (pull-or-build image, Gitea release API)
├── .forgejo/workflows/           # Forgejo mirror (same as Gitea)
├── .actrc                        # act config for local workflow validation
├── .dockerignore
├── .gitattributes
├── .gitignore
├── .gitlab-ci.yml                # GitLab mirror (docker-in-docker)
├── Jenkinsfile                   # Verification-only mirror
├── AI.md                         # This file
├── IDEA.md                       # Project plan (WHAT)
├── LICENSE.md
├── Makefile                      # build-env, build, validate, clean
├── README.md
├── SPEC.md                       # Project-specific rule overrides
└── renovate.json
```

All five CI providers (GitHub, Gitea, Forgejo, GitLab, Jenkins) are kept in lockstep: a change to targets, versions, or release steps is made in every one.

---

## Build image (`docker/Dockerfile.build`)

- Base `ubuntu:22.04`. Toolchains: zig (linux musl amd64/arm64), mingw-w64 (windows-amd64), llvm-mingw (windows-arm64), osxcross + MacOSX14.0 SDK (darwin), clang + lld with FreeBSD 14.5 sysroots (freebsd).
- `/opt/aria2-build/targets.sh` defines `set_target <os>-<arch>`, the single source of per-target compiler, host triplet, OpenSSL target, and flags. Both dependency builds and `build-aria2` source it.
- `/opt/build-deps.sh <target>` builds static zlib, OpenSSL, c-ares, libssh2, expat, SQLite into `/opt/deps/<target>`. One `RUN` layer per target for cache granularity.
- `/usr/local/bin/build-aria2 <os>-<arch> <version> <outdir>` downloads the upstream tarball, configures with `ARIA2_STATIC=yes` against `/opt/deps/<target>`, builds, strips, and writes `aria2c-<os>-<arch>[.exe]` to `<outdir>`.
- Version pins (ENV at top of file) are changed deliberately. OpenSSL stays on 3.x.

### aria2 configure flags

```
--enable-static --disable-shared --disable-nls
--with-openssl --with-libcares --with-libz --with-libexpat --with-libssh2 --with-sqlite3
--without-gnutls --without-libnettle --without-libgmp --without-libgcrypt
--without-libxml2 --without-libuv --without-jemalloc --without-tcmalloc
```

Per-target extras (`EXTRA_CONFIGURE`): `--without-wintls` on windows, `--without-appletls` on darwin.

### Linking notes

- Linux and FreeBSD link fully static (`-static`). Windows links mingw runtimes statically. macOS cannot link libSystem statically; only third-party libraries are static.
- OpenSSL is configured with `--openssldir=/etc/ssl` on linux/freebsd so the default CA lookup works on the host.
- Zig wrappers bake in `-static -L/opt/deps/<target>/lib` because autoconf's compiler test does not pass `LDFLAGS`.

---

## Versioning

- No version file in the repo. `get-version` resolves the latest upstream tag from `https://api.github.com/repos/aria2/aria2/releases/latest` (strip `release-`), or uses the `aria2_version` / `ARIA2_VERSION_OVERRIDE` input.
- Release tag: `v{version}`. Docker tags: `latest`, `{version}`, `YYMM`.

---

## Workflow triggers

| Workflow | Triggers |
|----------|----------|
| `build-env-image` | push touching `docker/Dockerfile.build`; quarterly cron; manual |
| `build-binaries` | push touching `docker/Dockerfile` or the workflow; successful `build-env-image` run; monthly cron; manual with optional version |
| `security` | push, PR, weekly cron, manual |

---

## Release assets

`aria2c-{os}-{arch}[.exe]` for all eight targets plus `checksums.txt` (SHA-256 of every binary). Linux, macOS, and FreeBSD binaries are marked executable before upload.

---

## Runtime image

`docker/Dockerfile`: `alpine:3` stage supplies `ca-certificates.crt`; final stage is `scratch` with the CA bundle and `/usr/local/bin/aria2c`, `WORKDIR /downloads`, `ENTRYPOINT ["/usr/local/bin/aria2c"]`. Built with context `.` for `linux/amd64,linux/arm64` after the release job copies the Linux binaries into the repo root.

---

## Security

- Every `uses:` in every workflow is pinned to a 40-char SHA (enforced by `security.yml § workflow-policy`); `pull_request_target` is forbidden.
- `security.yml` runs truffleHog on pushes/PRs and Trivy on the build (informational) and runtime (fail on CRITICAL/HIGH) images.
- Only `GITHUB_TOKEN` (and `GITEA_TOKEN` on Gitea/Forgejo) is used; no other secrets.

---

## Local use

```
make build-env
make build TARGET=linux-amd64 [VERSION=1.37.0]
make validate
```
