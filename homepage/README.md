# homepage service

Local build on top of the upstream image with tracked config baked in.

## Build

- `FROM ghcr.io/gethomepage/homepage:v2.4.0` — upstream image **is** the
  service (3rd-party dashboard); using it directly is allowed case-by-case.
- `COPY docker/*.yaml /app/config/` bakes the tracked config
  (`bookmarks.yaml`, `docker.yaml`, `services.yaml`, `settings.yaml`) into
  the image at build time.

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

Set via `.env` at the repo root (see `../.env.example`):

| Var | Required | Purpose |
| --- | --- | --- |
| `BASE_DOMAIN` | yes | Derives `VIRTUAL_HOST` / `LETSENCRYPT_HOST` (`homepage.<BASE_DOMAIN>`) and `HOMEPAGE_ALLOWED_HOSTS` |
| `NGINX_PROXY_NETWORK` | yes | Name of the existing external nginx-proxy network |
| `LETSENCRYPT_EMAIL` | yes | Contact email for Let's Encrypt certificates |
| `HOMEPAGE_VERSION` | yes | Pinned image version; must match Dockerfile `FROM`/`BASE_IMAGE` — bump together |
| `TZ` | no (default `UTC`) | Container timezone |

Derived (set in `docker-compose.yml`, not in `.env`): `VIRTUAL_HOST`,
`VIRTUAL_PORT=3000`, `LETSENCRYPT_HOST`, `HOMEPAGE_ALLOWED_HOSTS`.

## Why no entrypoint / PUID-PGID seeder

Stateless baked-config 3rd-party image: no volumes holding state, no file
ownership to fix, so no entrypoint or PUID/PGID handling is needed. The
container runs as the upstream default user.

## Security notes

- `/var/run/docker.sock` is mounted read-only (`:ro`) for container
  discovery via `docker.yaml`; a compromise of the dashboard still exposes
  the socket, so keep the image updated.
- No `ports:` published — only `expose: 3000`; ingress comes via the
  external nginx-proxy network only.
- Config is baked into the image, so a rebuild (`docker compose up -d
  --build`) is required for config changes to take effect.
