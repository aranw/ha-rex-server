# Changelog

## 0.1.3

- New `tailscale_hostname` option (default `ha-rex`): the Tailscale device
  name is now passed via `--tailscale-hostname`, so the host is
  `https://ha-rex.<tailnet>.ts.net` instead of `rex.<tailnet>.ts.net`,
  and configurable

## 0.1.2

- Fix container start: move `user-bundles.d` from `s6-rc.d/` to
  `/etc/s6-overlay/user-bundles.d/` (its s6-overlay location). Inside
  `s6-rc.d` s6-rc-compile expects a `type` file for it and the container
  died with "unable to read .../s6-rc.d/user-bundles.d/type"

## 0.1.1

- Fix `channel` option: list options are pipe-separated (`list(stable|preview|snapshot)`), so the Configuration tab now shows a real dropdown
- `extra_listen` and `tailscale_auth_key` are optional (`str?` / `password?`) and the extra listener is documented as advanced/dangerous in the UI
- `max_sessions` now defaults to 10 (0 still means unlimited)

## 0.1.0

Initial release: installs `rex` from the selected release channel
(stable/preview/snapshot) at every app start and runs `rex server run`
under s6-overlay, with optional Tailscale and plain-HTTP listeners.
