# Role Workflow Guide

This repository uses one standard lifecycle for all roles.

1. `assert`
2. `install`
3. `config`
4. `validate`

> **Note:** Not all phases are required in every role. For example, some roles may not need an `install` phase if no packages or dependencies are installed. Include only the phases relevant to your role.

## Phase Purpose

- `assert`
Validate role inputs before changing system state.
Fail early when required variables or value formats are invalid.

- `install`
Install packages, dependencies, users, directories, or service units required by the role.
This phase prepares everything needed before configuration is applied.

- `config`
Apply desired configuration and enforce final state for managed resources.
Keep tasks idempotent so reruns are safe.

- `validate`
Verify the resulting state matches expectations after configuration.
Use explicit checks and assertions so failures are clear and actionable.

## Recommended Task Structure

Use one `tasks/main.yml` entrypoint that imports phase files in order.

```yaml
---
- name: Assert
  ansible.builtin.import_tasks: assert.yml
  tags: [assert]

- name: Install
  ansible.builtin.import_tasks: install.yml
  tags: [install]

- name: Config
  ansible.builtin.import_tasks: config.yml
  tags: [config]

- name: Validate
  ansible.builtin.import_tasks: validate.yml
  tags: [validate]
```

## Tag Usage

Run specific phases during development or troubleshooting:

```bash
ansible-playbook ... --tags assert
ansible-playbook ... --tags install
ansible-playbook ... --tags config
ansible-playbook ... --tags validate
```

Run the full workflow by running the playbook with no tag filter.
