# roles/monitoring_authorized_key/README.md

Reference for the `monitoring_authorized_key` role.
Explains how the role installs an SSH authorized key for monitoring-style inter-host access.

## Purpose
- Allows a specific host (e.g. control) to SSH into other hosts to retrieve information
- Keeps monitoring/backup key management separate from bootstrap

## Variables

| Variable | Default | Required | Description |
|----------|---------|----------|-------------|
| `monitoring_authorized_user` | `bootstrap_user` or `root` | no | User to add the key to |
| `monitoring_authorized_key` | `""` | yes | SSH public key to authorize |

## Usage

```yaml
- hosts: dns
  roles:
    - monitoring/authorized_key
```

Set `monitoring_authorized_key` in group_vars for the target hosts:

```yaml
# group_vars/dns/vars.yml
monitoring_authorized_key: "ssh-ed25519 AAAA... user@control"
```

## Dependencies
None

## Author
tatbyte

## License
MIT
