# Ruust base images

Managed database engine base images for [Ruust](https://ruust.run), published to
the GitHub Container Registry as public packages.

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

Each starts `FROM` the official image, so we track upstream and add our own thin
layer on top. `linux/amd64` today (the fleet architecture); a two-arch manifest
can be added if arm64 hosts appear.

## Building

CI (`.github/workflows/build.yml`) builds every engine and version on push, on
demand, and monthly so upstream security patches flow on a predictable cadence.
Add a version by extending the build matrix and the control plane's
`DATABASE_ENGINES`.

## Licence

Apache-2.0 (see [`LICENSE`](LICENSE)). The upstream Postgres and Redis images
carry their own licences.
