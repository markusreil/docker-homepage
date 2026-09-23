# homepage service

Local build on top of the upstream image with tracked config baked in.

## Build

- `FROM ghcr.io/gethomepage/homepage:v2.4.0` — upstream image **is** the
  service (3rd-party dashboard); using it directly is allowed case-by-case.
- `COPY docker/*.yaml /app/config/` bakes the tracked config
  (`bookmarks.yaml`, `docker.yaml`, `services.yaml`, `settings.yaml`,
  `widgets.yaml`) into the image at build time.

Rebuild after any config change:

```sh
docker compose up -d --build
```

Stamp the build date (optional, defaults to `unknown`):

```sh
BUILD_DATE=$(date -u +%Y-%m-%dT%H:%M:%SZ) docker compose build
```

`BASE_IMAGE` is hardcoded in the Dockerfile (never in `.env`); `BUILD_DATE`
is passed as a build arg (`BUILD_DATE: ${BUILD_DATE:-unknown}` in compose)
into `ARG BUILD_DATE` / `ENV BUILD_DATE`.

Version bumps: `HOMEPAGE_VERSION` in `.env`, Dockerfile `FROM`, and `ENV
BASE_IMAGE` must all match — bump them together.

## Environment

Set via `.env` at the repo root (see `../env.example`):

| Var | Required | Purpose |
| --- | --- | --- |
| `BASE_DOMAIN` | yes | Derives the `x-hosts` anchor (`homepage.<BASE_DOMAIN>`) used for `VIRTUAL_HOST`, `ACME_HOST`, and `HOMEPAGE_ALLOWED_HOSTS` |
| `NGINX_PROXY_NETWORK` | no (default `web-proxy`) | Name of the existing external nginx-proxy network |
| `HOMEPAGE_VERSION` | yes | Pinned image version; must match Dockerfile `FROM`/`BASE_IMAGE` — bump together |
| `TZ` | no (default `UTC`) | Container timezone |
| `HOMEPAGE_GEN_SELF_SIGNED_CERT` | no (default `false`) | Self-signed TLS opt-in, honoured only by a LAN/self-signed proxy variant; leave `false` for an internet-facing cluster |

Derived (set in `docker-compose.yml`, not in `.env`): `VIRTUAL_HOST`,
`VIRTUAL_PORT=3000`, `ACME_HOST`, `GEN_SELF_SIGNED_CERT`,
`HOMEPAGE_ALLOWED_HOSTS`.

## Proxy contract

The service declares its complete, variant-agnostic proxy contract:
`VIRTUAL_HOST` and `ACME_HOST` (both the `x-hosts` anchor) plus
`GEN_SELF_SIGNED_CERT` wired from `HOMEPAGE_GEN_SELF_SIGNED_CERT` (default
`false`). An internet-facing cluster honours `ACME_HOST`; a LAN/self-signed
cluster honours `GEN_SELF_SIGNED_CERT`; each ignores the other. `VIRTUAL_PORT`
selects the exposed port `3000`. No `ports:` are published.

## Why no entrypoint / PUID-PGID seeder

Stateless baked-config 3rd-party image: no volumes holding state, no file
ownership to fix, so no entrypoint or PUID/PGID handling is needed. The
container runs as the upstream default user.

## Why no `homepage.*` labels

This service *is* the Homepage dashboard (COMPOSE-SPEC rule 11 N/A):
`homepage.*` labels are how downstream services opt in to discovery via
`docker.yaml`, so self-labelling would be self-referential. No `labels:`
in `docker-compose.yml` by intent; tag other containers (e.g.
`homepage.group`, `homepage.name`, `homepage.href`) to have them appear
on this dashboard.

## Security notes

- `/var/run/docker.sock` is mounted read-only (`:ro`) for container
  discovery via `docker.yaml`; a compromise of the dashboard still exposes
  the socket, so keep the image updated.
- No `ports:` published — only `expose: 3000`; ingress comes via the
  external nginx-proxy network only.
- Config is baked into the image, so a rebuild (`docker compose up -d
  --build`) is required for config changes to take effect.
