# roles/base_locale/README.md

Reference for the `base_locale` role.
Explains how the role ensures requested locales exist and configures the system default locale categories on Debian-family hosts during the base phase.

## Features
- Installs locale support packages with APT when needed
- Validates the requested locale variables before changes are made
- Ensures requested locales are available with `localedef` when they are missing
- Writes `/etc/default/locale` for Debian-family hosts
- Verifies both the generated locales and the configured default locale state

## Variables

| Variable | Default | Required | Description |
|----------|---------|----------|-------------|
| `base_locale_lang` | `en_US.UTF-8` | no | Locale assigned to `LANG` in the system locale configuration |
| `base_locale_lc_all` | `''` | no | Optional locale assigned to `LC_ALL`; leave empty to omit it |
| `base_locale_lc_time` | `''` | no | Optional locale assigned to `LC_TIME`; use it to control 12-hour vs 24-hour time formatting |
| `base_locale_packages` | `['locales']` | no | Package list installed with APT before locale generation and configuration |
| `base_locale_present` | `['{{ base_locale_lang }}']` | no | Locale names that must exist on the host before validation completes |

## Usage

The `base` role includes `base_locale` through meta dependencies.

Direct usage:

```yaml
- hosts: all
  become: true
  roles:
    - base_locale
```

Example variables:

```yaml
base_locale_lang: en_US.UTF-8
base_locale_lc_time: en_GB.UTF-8
base_locale_present:
  - en_US.UTF-8
  - en_GB.UTF-8
```

## Dependencies
None

## License
MIT

## Author
Tatbyte
