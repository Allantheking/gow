# syntax=docker/dockerfile:1.4
ARG BASE_APP_IMAGE
ARG BASE=ubuntu
ARG TAG=24.04

######################################
FROM lizardbyte/sunshine:latest AS sunshine-image

# hadolint ignore=DL3006
FROM ${BASE_APP_IMAGE} AS sunshine-base

ENV DEBIAN_FRONTEND=noninteractive

# Set ARG and ENV variables
ARG TARGETPLATFORM
ARG TARGETARCH
ARG TARGETVARIANT
ARG PGID=1001
ARG PUID=1001
ARG UNAME=lizard
ARG TZ="UTC"

ENV TARGETPLATFORM=${TARGETPLATFORM}
ENV PGID=${PGID}
ENV PUID=${PUID}
ENV UNAME=${UNAME}
ENV TZ=${TZ}
ENV HOME=/home/$UNAME

# Install dependencies
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN <<_DEPS
#!/bin/bash
set -e
apt-get update -y
apt-get install -y --no-install-recommends \
  libgbm1 \
  libvulkan1 \
  libvdpau1 \
  libnuma1 \
  libva-drm2 \
  libva-x11-2 \
  va-driver-all \
  libavahi-client3 \
  libcap2-bin
_DEPS

# Install architecture-specific dependencies
RUN <<_ARCH_DEPS
#!/bin/bash
set -e
if [[ "${TARGETARCH}" == "amd64" ]]; then
  apt-get install -y --no-install-recommends \
    i965-va-driver-shaders \
    intel-media-va-driver-non-free \
    libdrm-intel1
elif [[ "${TARGETARCH}" == "arm64" ]]; then
  apt-get install -y --no-install-recommends
fi
apt-get clean
rm -rf /var/lib/apt/lists/*
_ARCH_DEPS

# Install Sunshine using the official .deb package
RUN --mount=type=bind,from=sunshine-image,source=/sunshine.deb,target=/sunshine.deb <<_INSTALL_SUNSHINE
#!/bin/bash
set -e
apt-get update -y
apt-get install -y --no-install-recommends -f /sunshine.deb
apt-get clean
rm -rf /var/lib/apt/lists/*
_INSTALL_SUNSHINE

# Setup user
RUN <<_SETUP_USER
#!/bin/bash
set -e
groupadd -f -g "${PGID}" "${UNAME}"
useradd -lm -d ${HOME} -s /bin/bash -g "${PGID}" -u "${PUID}" "${UNAME}"
mkdir -p ${HOME}/.config/sunshine
ln -s ${HOME}/.config/sunshine /config
chown -R ${UNAME} ${HOME}
_SETUP_USER

USER ${UNAME}
WORKDIR ${HOME}

# Config files
COPY configs/sunshine.conf /cfg/sunshine.conf
COPY configs/apps.json /cfg/apps.json

COPY --chmod=777 scripts/startup.sh /opt/gow/startup-app.sh

# Port configuration taken from https://github.com/moonlight-stream/moonlight-docs/wiki/Setup-Guide#manual-port-forwarding-advanced
EXPOSE 47984-47990/tcp 48010 48010/udp 47998-48000/udp

# entrypoint
ENTRYPOINT ["/usr/bin/sunshine"]

ARG IMAGE_SOURCE
LABEL org.opencontainers.image.source=$IMAGE_SOURCE
