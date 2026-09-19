# homepage

A docker compose deployment of the [homepage](https://gethomepage.dev) dashboard, one instance per docker host, behind an [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy).

* discovers all docker UIs on the host via the docker socket
* vhost is the service name + `BASE_DOMAIN` (e.g. `homepage.example.com`)
* all env vars live in `.env` (tracked, no example file)
* compose fails fast when a required variable is unset

## Layout

| Path | Purpose |
| --- | --- |
| `docker-compose.yml` | Service definition, nginx-proxy wiring |
| `homepage/Dockerfile` | Builds the local image from `ghcr.io/gethomepage/homepage:latest` |
| `homepage/*.yaml` | Homepage config baked into the image at build time |
| `.env` | All required and optional variables |

## Prerequisites

* docker compose
* an existing nginx-proxy network (created via docker compose on the host running nginx-proxy)

## Setup

1. Edit `.env`:

   * `NGINX_PROXY_NETWORK` — name of the shared nginx-proxy network
   * `BASE_DOMAIN` — the root domain this host serves (e.g. `example.com`)
   * `LETSENCRYPT_EMAIL` — email for Let's Encrypt certificates (set `VIRTUAL_HOST` to `homepage.localhost` style if you do not want TLS)
   * `TZ` — optional, default `UTC`

2. Start it:

   ```sh
   docker compose up -d --build
   ```

3. Open `http://homepage.<BASE_DOMAIN>` (or `https://` once a certificate is issued).

## Configuration

Homepage config (`homepage/*.yaml`) is copied into the image during build, so config changes require a rebuild:

```sh
docker compose up -d --build
```

Container discovery is enabled in `homepage/docker.yaml` via the docker socket. Tag container UIs with `homepage.*` labels (e.g. `homepage.name`, `homepage.href`, `homepage.group`) to have them appear on the dashboard - see https://gethomepage.dev/configs/docker.