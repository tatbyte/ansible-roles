# Ansible Test Lab

This directory contains a minimal local harness for validating roles in this repository.

## Structure
- `ansible.cfg`: Test-specific Ansible configuration.
- `inventory/hosts.ini`: Test inventory.
- `inventory/group_vars/all.yml`: Shared variables for test hosts.
- `playbooks/bootstrap.yml`: Bootstrap phase using the standalone `bootstrap` role.
- `playbooks/base.yml`: Normal phase (connect as automation account).
- `playbooks/site.yml`: Full provisioning entry playbook that imports `bootstrap.yml` then `base.yml`.

## Usage
Run from repository root:

```sh
ANSIBLE_CONFIG=examples/ansible.cfg ansible-playbook examples/playbooks/site.yml
```

Or run directly from the `examples/` directory:

```sh
ansible-playbook playbooks/site.yml
```

Run only one phase when needed:

```sh
ANSIBLE_CONFIG=examples/ansible.cfg ansible-playbook examples/playbooks/site.yml --tags bootstrap
ANSIBLE_CONFIG=examples/ansible.cfg ansible-playbook examples/playbooks/site.yml --tags base
```

## Bootstrap Credentials
Bootstrap login values are intentionally stored in inventory for this example:

- `bootstrap_login_user`
- `bootstrap_login_password`
- `bootstrap_become_password`

They live under `[bootstrap:vars]` in `inventory/hosts.ini` and are consumed by `playbooks/bootstrap.yml`.

## Extending
- Add playbooks under `examples/playbooks/`.
- Update `examples/inventory/hosts.ini` and `examples/inventory/group_vars/` as needed.
- Keep this README aligned with any new test scenarios.
