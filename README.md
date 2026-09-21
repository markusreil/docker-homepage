# homepage

A docker compose deployment of the [homepage](https://gethomepage.dev) dashboard, one instance per docker host, behind an [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy).

* discovers all docker UIs on the host via the docker socket
* vhost is the service name + `BASE_DOMAIN` (e.g. `homepage.example.com`)
* all env vars live in `.env` (local, gitignored — copy from `env.example`)
* compose fails fast when a required variable is unset

## Layout

| Path | Purpose |
| --- | --- |
| `docker-compose.yml` | Service definition, nginx-proxy wiring |
| `homepage/Dockerfile` | Builds the local image from `ghcr.io/gethomepage/homepage:v2.4.0` |
| `homepage/docker/*.yaml` | Homepage config baked into the image at build time |
| `env.example` | Tracked source of truth for required and optional variables |
| `.env` | Local, gitignored copy of `env.example` with real values |

## Prerequisites

* docker compose
* an existing nginx-proxy network (created via docker compose on the host running nginx-proxy)

## Setup

1. Copy `env.example` to `.env` and edit `.env`:

   ```sh
   cp env.example .env
   ```

   * `NGINX_PROXY_NETWORK` — name of the shared nginx-proxy network
    * `BASE_DOMAIN` — the root domain this host serves (e.g. `example.com`)
    * `HOMEPAGE_VERSION` — pinned image version (e.g. `v2.4.0`; must match Dockerfile `FROM`)
   * `TZ` — optional, default `UTC`

2. Sanity-check, build, and start it:

   ```sh
   docker compose config  # fails fast on missing vars
   docker compose build
   docker compose up -d
   docker compose ps
   ```

   For stamped builds: `BUILD_DATE=$(date -u +%Y-%m-%dT%H:%M:%SZ) docker compose build`.

3. Open `http://homepage.<BASE_DOMAIN>` (or `https://` once a certificate is issued).

## Configuration

Homepage config (`homepage/docker/*.yaml`) is copied into the image during build, so config changes require a rebuild:

```sh
docker compose up -d --build
```

Container discovery is enabled in `homepage/docker/docker.yaml` via the docker socket. Tag container UIs with `homepage.*` labels (e.g. `homepage.name`, `homepage.href`, `homepage.group`) to have them appear on the dashboard - see https://gethomepage.dev/configs/docker.

For any `icon` field, three icon sets are available via their prefixes:

| Prefix | Icon set | Overview |
| --- | --- | --- |
| `mdi-` (e.g. `mdi-github`) | [Material Design Icons](https://pictogrammers.com/library/mdi/) | https://pictogrammers.com/library/mdi/ |
| `si-` (e.g. `si-github`) | [Simple Icons](https://simpleicons.org/) | https://simpleicons.org/ |
| `sh-` (e.g. `sh-adguard-home`) | [selfh.st/icons](https://selfh.st/icons/) | https://selfh.st/icons/ |

`mdi-` and `si-` icons default to SVG; the `sh-` prefix defaults to the PNG version (`sh-XX`, or force a version with `sh-XX.svg`/`.png`/`.webp`). A custom color can be appended to `mdi-`/`si-` icons as a hex suffix, e.g. `mdi-XX-#f0d453` - see https://gethomepage.dev/configs/services.