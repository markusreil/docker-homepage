# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- `CHANGELOG.md` and `AGENTS.md` per the compose project spec.
- `HOMEPAGE_GEN_SELF_SIGNED_CERT` opt-in (default `false`) so a LAN/self-signed
  or local-testing deployment can request a self-signed certificate without
  editing the compose file.
- Homepage dashboard widgets (`homepage/docker/widgets.yaml`) and a compact
  dashboard layout/title in `homepage/docker/settings.yaml`.

### Changed

- `HOMEPAGE_VERSION` is now a build arg passed from `.env` via compose, so
  the version is bumped in `.env` only; the Dockerfile `ARG` has no
  default, so plain `docker build` needs
  `--build-arg HOMEPAGE_VERSION=...`.

- Proxy contract migrated from the deprecated `LETSENCRYPT_HOST` to `ACME_HOST`
  (current upstream `acme-companion` naming).
- Hostname is now defined once in an `x-hosts` anchor and referenced for
  `VIRTUAL_HOST`, `ACME_HOST`, and `HOMEPAGE_ALLOWED_HOSTS`.
- `NGINX_PROXY_NETWORK` is optional and defaults to `web-proxy`.
- Documentation updated for the ACME variable naming and the self-signed
  opt-in.

### Deprecated

### Removed

- Deprecated `LETSENCRYPT_HOST` variable.

### Fixed

### Security

- Bumped `ghcr.io/gethomepage/homepage` from `v2.4.0` to `v2.7.0` to pick
  up fixed JS deps (`tar`, `brace-expansion`, `ip-address`).
- Added an `apk upgrade openssl libssl3 libcrypto3` layer in
  `homepage/Dockerfile` for OS CVEs left over in the upstream image.
- Removed `npm`/`npx`/`corepack` from the runtime image (it only ever runs
  `node server.js`); this drops npm's bundled `ip-address@10.5.0`, which
  no version bump could fix.
- Patched `undici` 8.11.2 to 8.12.3 and `systeminformation` 5.33.13 to
  5.33.19 in place from npm-registry tarballs (fixes CVE-2026-109869 and
  CVE-2026-116576, not yet fixed in upstream `v2.7.0`).
