# Project-specific rule overrides

This file wins over AI.md which wins over global CLAUDE.md.
Only rules that actively differ from the defaults are listed here.

---

## Exception 1 — Gitea/Forgejo/GitLab/Jenkins builds pull-or-build the build image

**Default rule:** CI binary builds run inside `ghcr.io/binmgr/aria2:build`.

**Override:** The Gitea, Forgejo, and GitLab pipelines run `docker pull ghcr.io/binmgr/aria2:build` and, if that fails, build `docker/Dockerfile.build` locally as `local/aria2:build`, then call `build-aria2` inside it.

**Rationale:** Self-hosted runners may not reach `ghcr.io`. Falling back to a local image build keeps those pipelines runnable without registry access, at the cost of a long first run.

---

## Exception 2 — Dependency source tarballs are not checksum-pinned

**Default rule:** Downloaded artifacts are verified against a known-good checksum.

**Override:** `docker/Dockerfile.build` downloads zlib, OpenSSL, c-ares, libssh2, expat, SQLite, and toolchain archives over HTTPS without pinned SHA-256 digests. `build-aria2` does the same for the aria2 tarball.

**Rationale:** A digest table for every upstream release is a recurring maintenance burden. The build image is rebuilt from scratch quarterly, and `checksums.txt` is published for every release asset.

**Accepted risk:** A compromised upstream tarball would not be caught before the binary is built.

---

## Exception 3 — Build and release are combined in `build-binaries.yml`

**Default rule:** Separate `build.yml` and `release.yml` workflows.

**Override:** One workflow builds all targets and publishes the release and runtime image. There is no intermediate test stage between build and release.

**Rationale:** Builds are deterministic for a given upstream version; a separate release gate adds no signal.

---

## Exception 4 — No language template in AI.md

**Default rule:** Projects copy the Go or Rust `AI.md` template.

**Override:** This is a CI/distribution project with no maintained Go, Rust, Node, or Python source. `AI.md` documents the build pipeline instead.

---

## Exception 5 — Runtime Dockerfile is `docker/Dockerfile` with the repo root as build context

**Default rule:** Dockerfile and build context live together under `docker/`.

**Override:** `docker/Dockerfile` is built with context `.` because the release job copies `aria2c-linux-amd64` and `aria2c-linux-arm64` into the repo root before `docker buildx build`. Do not move the binaries into `docker/`.

---

## Exception 6 — zig cc builds must disable UBSan

**Default rule:** Use the toolchain's default flags.

**Override:** Linux targets set `CFLAGS="-O2 -fno-sanitize=undefined"` in `targets.sh`. `zig cc` enables UBSan at `-O0`, which leaves `__ubsan_handle_*` symbols unresolved when linking the static OpenSSL and SQLite archives. Do not remove the flag. `build-aria2` also passes `-Wno-date-time` because aria2 uses `__DATE__`/`__TIME__` and zig's clang rejects it under `-Werror`.

---

## Exception 7 — Jenkins is verification-only

**Default rule:** All five CI providers publish releases.

**Override:** The Jenkinsfile builds and archives binaries but never publishes a release or image.

**Rationale:** Releases are owned by the hosted providers; Jenkins mirrors the build gates only.
