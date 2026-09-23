# Ruust base images

Base images for the [Ruust](https://ruust.run) Eggs that run our own software
rather than a customer build (the managed database engines and the Ollama private
LLM runtime), published to the GitHub Container Registry as public packages.

The fleet runs these instead of pulling the official images from Docker Hub
directly. Pulling `postgres`/`redis` anonymously on every host is rate limited (a
database Egg can fail to start with `toomanyrequests` at scale) and rides moving
upstream tags. Publishing them here gives us our own registry, a tag we move only
deliberately, and a place to add TLS, extensions (for example `pgvector`) and sane
defaults over time.

## Images

| Image | Tags | Base |
| --- | --- | --- |
| `ghcr.io/ruustrun/postgres` | `16`, `15`, `14` | `postgres:<version>` |
| `ghcr.io/ruustrun/redis` | `7`, `6` | `redis:<version>` |
| `ghcr.io/ruustrun/ollama` | `0` | `ollama/ollama:latest` |

Each starts `FROM` the official image, so we track upstream and add our own thin
layer on top. Ollama has no major-version line worth tracking, so its `0` tag
mirrors upstream `latest` and moves only when this repo's workflow runs. We ship
the Ollama runtime only, never models: those are pulled by the customer, under
their own licences. `linux/amd64` today (the fleet architecture); a two-arch manifest
can be added if arm64 hosts appear.

## Building

CI (`.github/workflows/build.yml`) builds every engine and version on push, on
demand, and monthly so upstream security patches flow on a predictable cadence.
Add a version by extending the build matrix and the control plane's
`DATABASE_ENGINES` (or `OLLAMA_IMAGE` for Ollama). A matrix entry may set `tag` to
publish under a different tag from the upstream version it builds from.

## Licence

Apache-2.0 (see [`LICENSE`](LICENSE)). The upstream Postgres and Redis images
carry their own licences.
