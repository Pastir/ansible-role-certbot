# pastir.certbot

Installs [certbot](https://certbot.eff.org/) on Ubuntu 24.04 (noble) and 26.04 (resolute), either from snap
(default) or from the Ubuntu apt package.

Issuing and renewing certificates is **never** managed: the role does not touch `/etc/letsencrypt` and does not
run `certbot certonly/renew`. Renewal is left to the timer shipped with the chosen install method; the role only
makes sure that timer is enabled and started.

## Requirements

- ansible-core >= 2.16, collection `community.general` (snap method) (`requirements.yml` in the role root)

## Install methods

### `snap` (default, behaviour of 1.0.0)
- removes the apt-packaged `certbot`, `python3-certbot-nginx`, `python3-certbot-apache` if
  `certbot_remove_apt_package: true` (default, kept for backward compatibility; set `false` to leave them);
- installs `snapd` and the `certbot` snap (classic, `certbot_snap_channel`);
- links `/snap/bin/certbot` to `certbot_link` (`force: true`);
- enables and starts `snap.certbot.renew.timer`.

### `apt`
- installs the `certbot` package (pinned with `certbot_apt_version`, if set) plus `certbot_apt_plugins`;
- with `certbot_apt_hold: true` (default) holds `certbot`, the plugins and `python3-certbot` (the core library,
  if installed) via `dpkg --set-selections`, so `apt upgrade` cannot move them;
- if `certbot_apt_version` is set and a different certbot version is installed, the role fails before changing
  anything unless `certbot_allow_upgrade: true` (then the hold is passed and the version is changed, up or down);
- enables and starts `certbot.timer` (shipped by the package);
- does not install, remove or touch snapd or any snaps.

Use `apt` on hosts that already run the packaged certbot with `certbot.timer`: running the role with the default
`snap` method there would replace it with the snap.

## Variables

| Variable | Default | Description |
|---|---|---|
| `certbot_install_method` | `snap` | `snap` or `apt` |
| `certbot_snap_channel` | `stable` | snap: channel of the certbot snap |
| `certbot_link` | `/usr/bin/certbot` | snap: where `/snap/bin/certbot` is linked |
| `certbot_remove_apt_package` | `true` | snap: remove the apt certbot and its nginx/apache plugins. Ignored by `apt` |
| `certbot_apt_version` | `""` | apt: version of the `certbot` package (`2.9.0-1`, `2.9.*`). Empty: any version |
| `certbot_apt_plugins` | `[]` | apt: extra packages, e.g. `[python3-certbot-nginx]` |
| `certbot_apt_hold` | `true` | apt: hold `certbot`, the plugins and `python3-certbot` |
| `certbot_allow_upgrade` | `false` | apt: allow changing an installed certbot version that differs from `certbot_apt_version` (passes the hold) |

## Check mode

`--check --diff` does not fail on a fresh host:
- snap: if `snapd` is not installed yet, installing the certbot snap is skipped (the `snap` command does not
  exist yet); the timer task is always skipped in check mode;
- apt: if the package would be installed, the `certbot.timer` task and holds of not-yet-installed packages are
  skipped (the unit and the packages do not exist yet).

## Example

```yaml
- hosts: web
  become: true
  roles:
    - role: pastir.certbot
      vars:
        certbot_install_method: apt
        certbot_apt_plugins: [python3-certbot-nginx]
```

## Existing servers

Pick the method that matches what is already installed (`snap list certbot` vs `dpkg -l certbot`), otherwise
the role switches the host from one to the other. Certificates in `/etc/letsencrypt` are not affected either way,
but the renewal hooks and timer of the old install are.

## Testing

`molecule test` (scenario `default`, podman, ubuntu 24.04 and 26.04): the **apt** method only — converge,
idempotence, check mode on the converged host, verify (`certbot --version`, plugin installed, snapd absent,
`certbot.timer` enabled and active, `certbot`/`python3-certbot`/plugin held).

The **snap** method is not covered by molecule: snapd does not run inside unprivileged containers. Verify it on a
VM with `--check --diff` first.

## Installation

```yaml
# requirements.yml
roles:
  - name: pastir.certbot
    src: git+git@github.com:Pastir/ansible-role-certbot.git
    version: main  # or a tag
# A role cannot install collections by itself: the project lists them (see requirements.yml in the role root).
collections:
  - name: community.general
```

```bash
ansible-galaxy install -r requirements.yml   # roles and collections
```

## License

MIT
