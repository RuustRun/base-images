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
| `ghcr.io/ruustrun/postgres` | `18`, `17`, `16`, `15`, `14`&nbsp;[^pg14] | `postgres:<version>` |
| `ghcr.io/ruustrun/redis` | `8`, `7`, `6` | `redis:<version>` |
| `ghcr.io/ruustrun/ollama` | `0` | `ollama/ollama:latest` |

[^pg14]: Still mirrored, but **retired from the control plane's offer list**: no new
Egg can be created on 14. It is kept here so the Eggs already running it receive
upstream patches until Postgres 14 reaches end of life on **2026-11-12**. Drop it from
the matrix after that date.

Each starts `FROM` the official image, so we track upstream and add our own thin
layer on top. Ollama has no major-version line worth tracking, so its `0` tag
mirrors upstream `latest` and moves only when this repo's workflow runs. We ship
the Ollama runtime only, never models: those are pulled by the customer, under
their own licences. `linux/amd64` today (the fleet architecture); a two-arch manifest
can be added if arm64 hosts appear.

## Building

`.github/workflows/build.yml` builds every engine and version on push, on demand,
and monthly so upstream security patches flow on a predictable cadence. A matrix
entry may set `tag` to publish under a different tag from the upstream version it
builds from.

### Adding a version, in this order

1. **Here first.** Add it to the build matrix and let CI publish the tag.
2. **Then the control plane**, in `DATABASE_ENGINES.versions` (or `OLLAMA_IMAGE`),
   newest first.

The order is not a style preference. The control plane builds the image ref straight
from the tag, so a version offered before its tag exists leaves a database Egg that
cannot pull. Think about `defaultVersion` separately: it is inherited by anything
that creates a database without naming a version, including the paired Postgres in a
game stack.

Existing Eggs keep the version they were created with. Changing a major needs a dump
and restore, so adding one only ever reaches newly created Eggs.

## Scanning and currency

Two jobs exist because publishing an image is not the same as knowing it is still
sound.

- **Trivy** runs on each image after build, before push (`build.yml`), at HIGH and
  CRITICAL, fixable only. Results go to the Security tab as SARIF so they keep their
  history. It is deliberately **not** a gate: these images are a passthrough of the
  official ones, so when upstream ships a fixable CVE we cannot patch it here, and
  failing the job would withhold the other versions' patches from the same rebuild
  while making nothing safer. A finding means upstream has a fix we are not carrying
  yet, so either the rebuild wants running sooner or that major is old enough to
  retire.
- **`upstream-watch.yml`** compares the majors on Docker Hub with this matrix weekly
  and opens a single issue when upstream is ahead. It never edits the matrix: whether
  to carry a new major, and whether it becomes the default, is a judgement. This is
  the job that should have caught us sitting two majors behind on Postgres, which a
  customer noticed first.

Note that a rebuilt tag reaching the registry is not the same as it reaching a host.
The agent re-pulls mutable tags on the next container create (RuustRun/agent#38);
before that it served whatever it had cached, forever.

## Licence

Apache-2.0 for this repo's own thin layer (see [`LICENSE`](LICENSE)). The upstream
images carry their own, and for Redis that is worth reading rather than assuming:

| Version | Licence |
| --- | --- |
| Postgres, all | PostgreSQL Licence (permissive) |
| Redis 7.2 and earlier (`6`) | BSD-3-Clause |
| Redis 7.4, which the `7` tag resolves to | RSALv2 **or** SSPLv1 |
| Redis 8 (`8`) | RSALv2, SSPLv1 **or** AGPLv3 |

Redis changed licence at 7.4, and both RSALv2 and SSPLv1 restrict providing the
software as a managed service. Redis 8 adds AGPLv3, which does not. That makes `8`
the licence-clear choice for a managed Redis Egg rather than merely the newest one.
Not legal advice: flagged here so the decision is made deliberately.
