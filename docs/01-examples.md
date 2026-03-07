# Test Lab Example

This document describes the example lab layout provided in `examples/`.

## Purpose

The `examples/` directory is an example environment that shows how to wire:

- inventory
- group variables
- playbooks
- `ansible.cfg`

Use it as a reference template for validating roles on your side.

## Example Files

- `examples/ansible.cfg`: Example Ansible configuration for local test runs.
- `examples/inventory/hosts.ini`: Example hosts and groups.
- `examples/inventory/group_vars/all.yml`: Example variables for all hosts.
- `examples/playbooks/base.yml`: Example playbook for the `base` role.
- `examples/playbooks/site.yml`: Example entry playbook that imports other playbooks.

## How to Use the Example

1. Copy or adapt the files in `examples/` to your own lab.
2. Replace inventory hosts and credentials with your environment values.
3. Update variables in `group_vars/all.yml` for your role inputs.
4. Run playbooks with the provided config pattern:

```bash
ANSIBLE_CONFIG=examples/ansible.cfg ansible-playbook examples/playbooks/site.yml
```

## Notes

- The lab content is intentionally simple and meant as an example baseline.
- Extend the inventory, vars, and playbooks to fit your own infrastructure and test scope.
