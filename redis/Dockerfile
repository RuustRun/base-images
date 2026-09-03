# Ruust managed Redis base image.
#
# See postgres/Dockerfile for the rationale: a pinned passthrough today (our own
# registry, a controlled tag, no Docker Hub pull rate limits), and the home for
# Redis config and hardening later. Built per major version via the VERSION build
# arg. British English, no em dashes.
ARG VERSION=7
FROM redis:${VERSION}

LABEL org.opencontainers.image.source="https://github.com/RuustRun/base-images"
LABEL org.opencontainers.image.description="Ruust managed Redis base image"
LABEL org.opencontainers.image.licenses="Apache-2.0"
