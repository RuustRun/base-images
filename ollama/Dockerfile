# Ruust Ollama base image (the private LLM Egg runtime).
#
# See postgres/Dockerfile for the rationale: our own registry, a tag we move only
# deliberately, and no Docker Hub pull rate limits on the fleet. This one matters
# more than most, because the image is large and an Ollama Egg is laid on customer
# hardware that may pull it cold.
#
# Unlike the database images there is no meaningful "major version" upstream, so we
# track upstream `latest` and publish it as our own `0` tag, moved when this workflow
# runs (on push, on demand, monthly). The control plane pins `ghcr.io/ruustrun/ollama:0`
# (OLLAMA_IMAGE in lib/eggs.ts).
#
# We ship the RUNTIME only, never models: the customer pulls the models they want and
# accepts their licences. British English, no em dashes.
ARG VERSION=latest
FROM ollama/ollama:${VERSION}

LABEL org.opencontainers.image.source="https://github.com/RuustRun/base-images"
LABEL org.opencontainers.image.description="Ruust Ollama base image (private LLM Egg runtime)"
LABEL org.opencontainers.image.licenses="Apache-2.0"
