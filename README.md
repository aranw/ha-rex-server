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

Then install **Rex Server** from the store. Prebuilt images are published as
a single multi-arch image, `ghcr.io/aranw/ha-rex-server:<version>` (amd64 and
arm64 in one manifest list).

For development, copy the `rex-server/` folder into `local_apps` on your
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
access). See [rex-server/DOCS.md](rex-server/DOCS.md) for details and the
security model.

## Repository layout

```
rex-server/            the app (config.yaml, Dockerfile, rootfs, translations)
.github/workflows/     builds and publishes the images with home-assistant/builder
```

## License

MIT — see [LICENSE](LICENSE).
