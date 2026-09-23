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
