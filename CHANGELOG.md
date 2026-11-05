# Changelog

## 1.1.0 — 2026-11

### Added
- `certbot_install_method: snap | apt` (default `snap`, unchanged behaviour). The `apt` method installs the
  `certbot` package (`certbot_apt_version`, `certbot_apt_plugins`) and enables `certbot.timer`; it never touches
  snapd or snaps.
- `certbot_apt_hold` (default `true`): holds `certbot`, the plugins and `python3-certbot`.
- `certbot_allow_upgrade` (default `false`): with a pinned `certbot_apt_version` that differs from the installed
  one the role fails before changing anything unless this is `true`.
- Variable validation (`tasks/assert.yml`).
- Molecule scenario `default` for the `apt` method on Ubuntu 24.04 and 26.04. The snap method stays untested
  in containers.

### Changed
- Tasks split into `tasks/snap.yml` and `tasks/apt.yml`; tags `certbot_install` / `certbot_config` added.
- `certbot_remove_apt_package` now applies to the snap method only (default `true`, same as before).

### Fixed
- `--check` failed on hosts without snapd: installing the certbot snap is now skipped in check mode while the
  `snap` command does not exist yet.

## 1.0.0 — 2026-11

First version.
