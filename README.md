# ha-rex-server

A [Home Assistant app](https://contributors.home-assistant.io/apps/) that
installs and runs a [Rex](https://www.superlogical.com/rex/) terminal
server, so Rex clients (Rex for Mac, `rex` CLI) can open terminal sessions
on your Home Assistant machine.

## Install

In Home Assistant, go to **Settings > Apps > App store > ⋮ > Repositories**
and add:

```
https://github.com/aranw/ha-rex-server
```

or click this badge to open the dialog with the URL pre-filled:

[![Add repository](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Faranw%2Fha-rex-server)

Then install **Rex Server** from the store. Prebuilt images are published as
a single multi-arch image, `ghcr.io/aranw/ha-rex-server:<version>` (amd64 and
arm64 in one manifest list).

For development, copy the `rex_server/` folder into `local_apps` on your
Home Assistant instead; it shows up under **Local apps** and the Supervisor
builds it from source.

## How it works

The image carries Rex's apk signing key but no `rex` binary. At every app
start a setup service installs/upgrades `rex` from the release channel
chosen in the app's Configuration tab (`stable`, `preview`, or `snapshot`),
then an s6-overlay service runs `rex server run` in the foreground.

Connect with Tailscale (recommended: set a one-time auth key, then add
`https://rex.<tailnet>.ts.net` as a remote host) or with a plain HTTP
listener on the LAN (off by default — anyone who can reach it gets full
access). See [rex_server/DOCS.md](rex_server/DOCS.md) for details and the
security model.

## Repository layout

```
repository.yaml        required: marks this repo as a Home Assistant app repository
rex_server/            the app (config.yaml, Dockerfile, rootfs, translations)
.github/workflows/     builds and publishes the multi-arch image with Docker Buildx
```

## License

MIT — see [LICENSE](LICENSE).
