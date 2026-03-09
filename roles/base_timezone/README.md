# roles/base_timezone/README.md

Reference for the `base_timezone` role.
Explains how the role enforces the system timezone on Debian-family hosts as part of the base phase.

## Features
- Installs the timezone database package with APT before timezone configuration
- Validates the requested timezone value
- Configures `/etc/localtime` to the requested timezone
- Optionally writes `/etc/timezone` on Debian-family systems
- Verifies the configured timezone state after changes

## Variables

| Variable | Default | Required | Description |
|----------|---------|----------|-------------|
| `base_timezone_packages` | `['tzdata']` | no | Package list installed with APT to provide timezone data on the host |
| `base_timezone_name` | `Etc/UTC` | no | IANA timezone name to enforce on the host |

## Usage

The `base` role now includes `base_timezone` through meta dependencies.

Direct usage:

```yaml
- hosts: all
  become: true
  roles:
    - base_timezone
```

Example variable:

```yaml
base_timezone_name: America/Toronto
```

## Dependencies
None

## License
MIT

## Author
Tatbyte
