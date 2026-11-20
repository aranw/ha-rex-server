# Changelog

## 0.1.1

- Fix `channel` option: list options are pipe-separated (`list(stable|preview|snapshot)`), so the Configuration tab now shows a real dropdown
- `extra_listen` and `tailscale_auth_key` are optional (`str?` / `password?`) and the extra listener is documented as advanced/dangerous in the UI
- `max_sessions` now defaults to 10 (0 still means unlimited)

## 0.1.0

Initial release: installs `rex` from the selected release channel
(stable/preview/snapshot) at every app start and runs `rex server run`
under s6-overlay, with optional Tailscale and plain-HTTP listeners.
