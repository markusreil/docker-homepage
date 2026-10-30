# No default: the version comes exclusively from compose (.env), which
# fails fast if HOMEPAGE_VERSION is unset. Plain `docker build` needs
# --build-arg HOMEPAGE_VERSION=...
ARG HOMEPAGE_VERSION
FROM ghcr.io/gethomepage/homepage:${HOMEPAGE_VERSION}
# Re-declared: an ARG before FROM is out of scope after it.
ARG HOMEPAGE_VERSION
ARG BUILD_DATE=unknown
ENV BASE_IMAGE=ghcr.io/gethomepage/homepage:${HOMEPAGE_VERSION}
ENV BUILD_DATE=${BUILD_DATE}
# Patch OS-level CVEs left over in the upstream image (e.g. openssl
# libssl3/libcrypto3). JS deps (tar, brace-expansion, ip-address) ride with
# the upstream release, so prefer a version bump over patching them here.
# Runtime needs only `node server.js` (plus wget for the healthcheck): drop
# npm/npx/corepack, whose bundled deps (e.g. ip-address@10.5.0) otherwise
# linger as scanner findings in an appliance image that never runs them.
RUN apk update && apk upgrade --no-cache openssl libssl3 libcrypto3 && rm -rf /var/cache/apk/* \
  && rm -rf /usr/local/lib/node_modules/npm /usr/local/lib/node_modules/corepack \
    /usr/local/bin/npm /usr/local/bin/npx /usr/local/bin/corepack
# Patch JS CVEs not yet fixed upstream by replacing the vulnerable packages
# in place with fixed tarballs from the npm registry. The .pnpm directory
# names keep their old version suffix on purpose so existing symlinks stay
# intact; package.json + contents are genuinely the fixed releases.
RUN cd /tmp \
  && wget -q https://registry.npmjs.org/undici/-/undici-8.12.3.tgz \
  && wget -q https://registry.npmjs.org/systeminformation/-/systeminformation-5.33.19.tgz \
  && rm -rf /app/node_modules/.pnpm/undici@8.11.2/node_modules/undici \
  && mkdir -p /app/node_modules/.pnpm/undici@8.11.2/node_modules/undici \
  && tar xzf undici-8.12.3.tgz -C /app/node_modules/.pnpm/undici@8.11.2/node_modules/undici --strip-components=1 \
  && rm -rf /app/node_modules/.pnpm/systeminformation@5.33.13/node_modules/systeminformation \
  && mkdir -p /app/node_modules/.pnpm/systeminformation@5.33.13/node_modules/systeminformation \
  && tar xzf systeminformation-5.33.19.tgz -C /app/node_modules/.pnpm/systeminformation@5.33.13/node_modules/systeminformation --strip-components=1 \
  && rm -f undici-8.12.3.tgz systeminformation-5.33.19.tgz
COPY docker/*.yaml /app/config/
