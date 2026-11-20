# Base image. Overridable at build time:
#   docker build --build-arg BUILD_FROM=ghcr.io/hassio-addons/base:19 .
# or via the `base_image` input of the build workflow. The base must be a
# multi-arch manifest (linux/amd64 + linux/arm64), Alpine-based, with
# s6-overlay v3 (entrypoint /init, init: false in config.yaml) and bashio
# (`#!/command/with-contenv bashio`) - the rex-install service uses apk.
ARG BUILD_FROM=ghcr.io/home-assistant/base:3.24
FROM ${BUILD_FROM}

# curl + CA certificates: needed at build time (signing key download) and at
# runtime (channel check and apk in the rex-install service).
RUN apk add --no-cache curl ca-certificates

# Rex's apk signing key, baked into the image, so the runtime `apk add rex`
# is signature-verified. The key file name is pinned by Rex's own installer
# (APK_KEY_NAME in https://pkgs.superlogical.com/rex/install.sh).
ARG REX_APK_KEY_NAME=hello@superlogical.com-33cd23a8.rsa.pub
RUN curl -fsSL "https://pkgs.superlogical.com/rex/keys/${REX_APK_KEY_NAME}" \
      -o "/etc/apk/keys/${REX_APK_KEY_NAME}"

COPY rootfs /

# s6-overlay executes service scripts directly, so they must be executable.
# Keep LF line endings (see .gitattributes at the repository root).
RUN chmod a+x \
      /etc/s6-overlay/s6-rc.d/rex-install/run \
      /etc/s6-overlay/s6-rc.d/rex-server/run \
      /etc/s6-overlay/s6-rc.d/rex-server/finish
