# pastir.certbot

Installs [certbot](https://certbot.eff.org/) from snap (classic) and links it to `/usr/bin/certbot`.
Issuing and renewing certificates is **not** managed: the snap's `snap.certbot.renew.timer` takes care of renewal
(the role only makes sure the timer is enabled). The apt-packaged certbot, which conflicts with the snap, is removed.

## Variables

| Variable | Default | Description |
|---|---|---|
| `certbot_snap_channel` | `stable` | |
| `certbot_link` | `/usr/bin/certbot` | |
| `certbot_remove_apt_package` | `true` | |

## Testing

Not covered by molecule: snapd does not run inside unprivileged containers. Verify on a VM with
`--check --diff` first.

## Installation

```yaml
# requirements.yml
roles:
  - name: pastir.certbot
    src: git+git@github.com:Pastir/ansible-role-certbot.git
    version: main  # or a tag
```

```bash
ansible-galaxy install -r requirements.yml
```

## License

MIT
