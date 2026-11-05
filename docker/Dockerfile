# =============================================================================
# aria2 Runtime Image
# =============================================================================
# Minimal image containing only the static aria2c binary and CA certificates.
# Multi-arch: linux/amd64, linux/arm64
#
# Usage:
#   docker run --rm ghcr.io/binmgr/aria2:latest --version
# =============================================================================

FROM alpine:3 AS certs
RUN apk add --no-cache ca-certificates

FROM scratch

ARG TARGETARCH

COPY --from=certs /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY aria2c-linux-${TARGETARCH} /usr/local/bin/aria2c

WORKDIR /downloads
ENTRYPOINT ["/usr/local/bin/aria2c"]
CMD ["--help"]
