# Ruust managed Postgres base image.
#
# Starts from the official Postgres image and adds a thin Ruust layer. Today that
# layer is just labels: the image is a pinned passthrough so the fleet pulls from
# our own registry (avoiding Docker Hub pull rate limits) on a tag we move only
# deliberately. This is the home for TLS, extensions (pgvector and friends) and
# sane defaults as we harden the managed database offering.
#
# Built per major version via the VERSION build arg (see .github/workflows/build.yml),
# kept in step with DATABASE_ENGINES in the Ruust control plane. British English,
# no em dashes.
ARG VERSION=16
FROM postgres:${VERSION}

LABEL org.opencontainers.image.source="https://github.com/RuustRun/base-images"
LABEL org.opencontainers.image.description="Ruust managed Postgres base image"
LABEL org.opencontainers.image.licenses="Apache-2.0"
