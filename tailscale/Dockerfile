ARG BUILD_FROM=ghcr.io/hassio-addons/base/amd64:18.1.1
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Setup base
ARG BUILD_ARCH=amd64
ARG TAILSCALE_VERSION="v1.96.4"
# THE PINS ARE THE PROBLEM, NOT THE FROZEN TAG.
#
# The base writes EXACT version constraints into /etc/apk/world (libcrypto3,
# libssl3, musl, curl, bash, jq, tzdata). A world constraint is precisely what
# apk must not upgrade past, so a plain `apk upgrade` is a no-op for exactly
# the packages that carry the CVEs — `apk policy libcrypto3` reports the fixed
# version as available and apk still refuses it. Relax the constraints first,
# then upgrade. The Alpine BRANCH stays pinned, so this takes patch-level
# security updates and never a distro jump.
#
# Measured on the pinned base, 2026-08-26: 2 fixable HIGH before (libcrypto3 +
# libssl3 3.5.7-r0, CVE-2026-14456), 0 after. The identical two on our own
# armv7 base — and "use a newer base" is not an option that exists for armv7,
# which upstream froze at 18.2.1 when 19.0.0 dropped the arch.
#
# hadolint ignore=DL3017,DL3018
RUN sed -i 's/=[0-9][^[:space:]]*//g' /etc/apk/world \
  && apk --no-cache upgrade

RUN \
    apk add --no-cache \
        ethtool \
        ipcalc \
        iproute2 \
        iptables \
        nginx \
        coreutils \
        networkmanager \
    \
    && ln -sf /sbin/xtables-nft-multi /sbin/ip6tables \
    && ln -sf /sbin/xtables-nft-multi /sbin/iptables \
    \
    && if [ "${BUILD_ARCH}" = "aarch64" ]; then ARCH="arm64"; fi \
    && if [ "${BUILD_ARCH}" = "armv7" ]; then ARCH="arm"; fi \
    && if [ "${BUILD_ARCH}" = "amd64" ]; then ARCH="amd64"; fi \
    \
    && curl -L -s \
        "https://pkgs.tailscale.com/stable/tailscale_${TAILSCALE_VERSION#v}_${ARCH}.tgz" \
        | tar zxvf - -C /opt/ --strip-components 1 \
    \
    && rm -f -r \
        /opt/systemd

# Copy root filesystem
COPY rootfs /

HEALTHCHECK \
    CMD healthcheck

# S6 Overlay stage 2 hook
ENV S6_STAGE2_HOOK=/etc/s6-overlay/scripts/stage2_hook.sh

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Community Add-ons" \
    org.opencontainers.image.authors="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://addons.community" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
