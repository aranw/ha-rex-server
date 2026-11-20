# ha-rex-server

A [Home Assistant app](https://contributors.home-assistant.io/apps/) that
installs and runs a [Rex](https://www.superlogical.com/rex/) terminal
server, so Rex clients (Rex for Mac, the `rex` CLI on another machine) can
open terminal sessions on your Home Assistant machine.

## Features

- **Release channel selection** — pick `stable`, `preview` (default), or
  `snapshot` in the app's Configuration tab; the app installs/upgrades `rex`
  from that channel at every start
- **Tailscale listener** (recommended) — the server joins your tailnet as its
  own device, `https://ha-rex.<tailnet>.ts.net` by default; the device name
  is configurable (*Tailscale hostname*)
- **Optional plain-HTTP listener** on the LAN/VPN (port 7297, off by
  default, requires an explicit opt-in) — anyone who can reach it gets full
  access, so prefer Tailscale
- **Max sessions** limit (default 10; 0 = unlimited)
- Persistent state in `/data` (included in backups): session state and the
  Tailscale login

## Install

In Home Assistant, go to **Settings > Apps > App store > ⋮ > Repositories**
and add:

```
https://github.com/aranw/ha-rex-server
```

or click this badge to open the dialog with the URL pre-filled:

[![Add repository](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Faranw%2Fha-rex-server)

Then install **Rex Server** from the app store. Prebuilt images are published
as a single multi-arch image, `ghcr.io/aranw/ha-rex-server:<version>` (amd64
and arm64 in one manifest list), tagged with the app version from
`rex_server/config.yaml`.

For development, copy the `rex_server/` folder into `local_apps` on your
Home Assistant instead (via the Samba share or Terminal & SSH app); it shows
up under **Local apps** and the Supervisor builds it from source.

## Connecting a client

1. **Tailscale (recommended).** In the app's Configuration tab, set
   *Tailscale auth key* to a one-time Tailscale auth key (and optionally
   *Tailscale hostname*, default `ha-rex`). The tailnet needs MagicDNS and
   HTTPS certificates enabled. Then in Rex for Mac choose
   **Add Remote Host…** and enter `https://ha-rex.<tailnet>.ts.net`, or from
   a CLI: `rex -S https://ha-rex.<tailnet>.ts.net ls`.
   The login persists in `/data/rex`, so the key is needed only once.
2. **Plain HTTP on the LAN/VPN.** Set *Extra network listener* to `:7297`,
   enable *Allow anonymous access*, and enable port 7297 in the app's Network
   settings. Then connect with `rex -S http://homeassistant.local:7297`.
   **Anyone who can reach that port gets full access to the server** — only
   do this on a network you completely control.

## How it works

The image carries Rex's apk signing key but no `rex` binary. At every app
start, an s6-overlay oneshot (`rex-install`) installs/upgrades `rex` from the
selected channel's Alpine package repository, then a longrun (`rex-server`)
runs:

```
rex server run --listen unix:///run/rex/server.sock --state-dir /data/rex ...
```

Changing any option requires an app restart; that restart is also what
applies a new channel or a newer `rex` build from the same channel.

See [rex_server/DOCS.md](rex_server/DOCS.md) for the full options table, the
security model (protection mode, container-only terminals, why the plain
listener is dangerous), and development notes.

## Options

| Option | Default | Description |
| --- | --- | --- |
| `channel` | `preview` | Rex release channel: `stable`, `preview`, `snapshot` (dropdown). Applied on restart. |
| `tailscale_auth_key` | empty (optional) | One-time Tailscale auth key for the Tailscale listener. |
| `tailscale_hostname` | `ha-rex` (optional) | Tailscale device name; reachable as `https://<hostname>.<tailnet>.ts.net`. |
| `extra_listen` | empty (optional) | Extra network listener, e.g. `:7297`. Requires `allow_anonymous` + port 7297 enabled. Dangerous on an open network. |
| `allow_anonymous` | `false` | Lets the extra listener accept unidentified callers. |
| `max_sessions` | `10` | Max concurrent sessions; 0 = unlimited. |

## Repository layout

```
repository.yaml        required: marks this repo as a Home Assistant app repository (name shown in the store)
rex_server/            the app (config.yaml, Dockerfile, rootfs/, translations/, DOCS.md)
.github/workflows/     builds and publishes the multi-arch image with Docker Buildx
```

## Releases and custom base images

Push a tag `vX.Y.Z` (matching `version` in `rex_server/config.yaml`) to build
and publish `ghcr.io/aranw/ha-rex-server:X.Y.Z`. The build workflow also has a
`base_image` input (Run workflow) to build against a different base image —
see [rex_server/DOCS.md](rex_server/DOCS.md#custom-base-image) for the
constraints (multi-arch, Alpine, s6-overlay v3, bashio).

## License

MIT — see [LICENSE](LICENSE).
