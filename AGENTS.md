# AGENTS.md

Ongoing rules for working in this repo. See `README.md` and
`homepage/README.md` for detail; this is a summary, not a copy.

## Configuration

- All per-deployment configuration lives in `.env` (hidden, never tracked).
  Track `env.example` instead, keep both files in the same order/shape, and
  start with `cp env.example .env`.
- Group variables: global first, then one `# --- <service> ---` section per
  compose service; every variable keeps its own comment.
- Required variables use `${VAR:?...}` so `docker compose config` fails fast;
  optional ones use `${VAR:-default}` and are documented as optional. Never
  replace a `:?` guard with a silent default.
- `BASE_IMAGE` is hardcoded in each Dockerfile (build-time only); `BUILD_DATE`
  is a build arg (`${BUILD_DATE:-unknown}`) stamped with
  `BUILD_DATE=$(date -u +%Y-%m-%dT%H:%M:%SZ) docker compose build`. Neither
  belongs in `.env` / `env.example`.

## Compose

- Single `docker-compose.yml`; set the project `name:` once and never use
  `container_name:`.
- Hostnames come from the `x-hosts` anchors at the top, derived from
  `BASE_DOMAIN` — never copy-pasted literals.
- No `ports:`; services join the external proxy network
  (`NGINX_PROXY_NETWORK`, default `web-proxy`, `external: true`) and advertise
  via `expose:`.
- Proxied services declare their **complete** contract: `VIRTUAL_HOST` always,
  `VIRTUAL_PORT`/`VIRTUAL_PROTO` where the defaults do not fit, and for HTTPS
  `ACME_HOST` plus `GEN_SELF_SIGNED_CERT` wired from an optional `.env`
  variable (default `false`). Use the `ACME_*` spelling, never
  `LETSENCRYPT_*` (the sole exception is the `LETSENCRYPT_TEST` staging
  toggle).
- Stateful data uses standard named volumes, not host bind mounts (config
  overlays, the docker socket, and devices are the documented exceptions).
  `restart: unless-stopped`.

## Images / entrypoints

- Prefer building from source on an `alpine` base; use an upstream image only
  when the image *is* the service, documented in the service README.
- Custom entrypoints (only where needed) seed config on first start only and
  drop privileges via `PUID`/`PGID`; chown config dirs recursively, data dirs
  only at the root. Document why when there is no entrypoint.

## Docs / changelog

- Keep `CHANGELOG.md` (Keep a Changelog) current in `## [Unreleased]` as
  changes are made; do not create release sections unprompted.
- Set Homepage labels (`homepage.group`, `homepage.name`, `homepage.icon`,
  `homepage.href`, `homepage.description`) on services that should appear on
  the dashboard.
- Update `README.md` and the service README when behavior changes.

## Verify

```sh
cp env.example .env   # then edit
docker compose config # fails fast on missing vars
docker compose build
docker compose up -d
docker compose ps
```
