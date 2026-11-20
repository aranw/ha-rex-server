# Rex Server

This app runs a [Rex](https://www.superlogical.com/rex/) terminal server
inside Home Assistant. Rex clients (Rex for Mac, the CLI on another machine)
connect to it and get interactive terminal sessions.

## How it works

The container image does not bake in a `rex` binary. It bakes in Rex's apk
signing key, and at every app start a setup service installs/upgrades `rex`
from the **release channel** you picked in the app's Configuration tab
(`stable`, `preview`, or `snapshot`), using Rex's Alpine package repository
(`pkgs.superlogical.com`). A second service then runs the server in the
foreground under s6-overlay:

```
rex server run --listen unix:///run/rex/server.sock --state-dir /data/rex ...
```

- The Unix socket (`/run/rex/server.sock`) is runtime-only and not backed up.
- The state directory `/data/rex` is persistent and included in backups
  (it holds session state and the Tailscale login, not the auth key itself).
- Changing any option requires an app restart; that restart is also what
  applies a new channel or a newer `rex` build from the same channel.

## Connecting clients

Rex identifies callers only by how they connect. Pick one:

1. **Tailscale (recommended).** Put a one-time Tailscale auth key in the
   *Tailscale auth key* option. The server joins your tailnet as the device
   `rex.<tailnet>.ts.net`. In Rex for Mac choose **Add Remote Host…** and
   enter `https://rex.<tailnet>.ts.net`, or from a CLI:
   `rex -S https://rex.<tailnet>.ts.net ls`
   The tailnet must have MagicDNS and HTTPS certificates on. The server
   keeps its login in `/data/rex`, so the key is needed only once.
2. **Plain HTTP on the LAN/VPN.** Set *Extra listen address* to `:7297`,
   enable *Allow anonymous access*, and enable port 7297 in the app's
   Network settings. Then connect with `rex -S http://homeassistant.local:7297`.
   **Anyone who can reach that port gets full access to the server** — only
   do this on a network you completely control.
3. **SSH** is not set up inside the app container; use option 1 or 2.

## Security model

- A connection to the Rex server can run any command the server can run.
  Keep Protection mode **on** and do not add `full_access`, `docker_api`, or
  `host_pid` to this app.
- Terminals run **inside the app container**, not on the Home Assistant OS
  host (`host_pid` does not work with s6-overlay anyway). The container sees
  only what the app is mapped to (its own `/data`).
- Keep the 7297 port disabled and prefer Tailscale; Rex's plain-HTTP mode
  is unauthenticated by design.

## Options

| Option | Default | Description |
| --- | --- | --- |
| `channel` | `preview` | Rex release channel to install: `stable`, `preview`, `snapshot`. Applied on restart. If the channel is not published yet (e.g. `stable` before the first stable release), the app stops with a clear log message. |
| `tailscale_auth_key` | empty | One-time Tailscale auth key for the built-in Tailscale listener. |
| `extra_listen` | empty (optional) | Extra address the server also listens on for remote clients, e.g. `:7297`. Requires `allow_anonymous` and enabling port 7297 in the Network section. Anyone who can reach it gets full access. |
| `allow_anonymous` | `false` | Allows the extra listener to accept unidentified callers. Dangerous on an open network. |
| `max_sessions` | `10` | Max concurrent sessions; 0 = unlimited. |

## Development

Copy `rex_server/` into `local_apps` on your Home Assistant (via the Samba
share or Terminal & SSH app); it appears under **Local apps**. Bump `version`
in `config.yaml` to make the Supervisor rebuild. Release images are built
and pushed as one multi-arch manifest, `ghcr.io/aranw/ha-rex-server:<version>`,
by `.github/workflows/build.yaml` with Docker Buildx.
