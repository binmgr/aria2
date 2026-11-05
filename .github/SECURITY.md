# Security Policy

## Supported Versions

This repository builds and publishes the latest stable aria2 release. Only the most recent published release receives security updates — older tagged releases are not patched.

| Version | Supported |
|---------|-----------|
| Latest published release | Yes |
| Older releases | No — rebuild from `main` |

## Reporting a Vulnerability

If you discover a security issue with the build pipeline, container image, or published binaries, please report it privately:

- Use GitHub's **Private vulnerability reporting** on this repository, or
- Email the maintainers at `security@binmgr.invalid`

**Do not open a public issue for security reports.**

You will receive an acknowledgement within 48 hours. Confirmed critical issues will be patched within 14 days; the fix will be released in a new tagged build with the issue referenced in the release notes.

## Disclosure Timeline

This project follows coordinated disclosure. Once a fix is released, security advisories will be published via GitHub Security Advisories with full details and credit to the reporter (unless anonymity is requested).

## Out of Scope

- Vulnerabilities in upstream aria2 itself — report those to the [aria2 project](https://github.com/aria2/aria2/security).
- Vulnerabilities in bundled libraries (OpenSSL, libssh2, c-ares, expat, SQLite, zlib) — report to their upstream maintainers.
- Misuse of the binaries to download content you do not have the rights to access.
