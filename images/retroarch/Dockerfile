ARG BASE_APP_IMAGE

# hadolint ignore=DL3006
FROM ${BASE_APP_IMAGE}

ARG DEBIAN_FRONTEND=noninteractive
ARG REQUIRED_PACKAGES=" \
    wget p7zip-full \
    retroarch \
    libretro-core-info libretro-frontend \
    libusb-1.0-0 libglu1-mesa libaio1t64 libaio-dev"

RUN apt-get update && \
    apt-get install -y --no-install-recommends software-properties-common gpg-agent && \
    # Install retroarch \
    apt-get install -y --no-install-recommends $REQUIRED_PACKAGES && \
    # Cleanup \
    apt-get remove -y software-properties-common gpg-agent && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/*

COPY configs/retroarch.cfg /cfg/retroarch.cfg
COPY --chmod=777 scripts/startup.sh /opt/gow/startup-app.sh

ENV XDG_RUNTIME_DIR=/tmp/.X11-unix

ARG IMAGE_SOURCE
LABEL org.opencontainers.image.source $IMAGE_SOURCE
