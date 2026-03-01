# _common

Shared “framework” files used by all roles.

This folder provides a **task flow orchestrator** that standardizes how roles run their tasks in ordered phases (assert → install → config → validate), with:

- **Profiles** (categories) like `full`, `config`, `guard` (+ dev variants)
- Per-role **enable/disable** switch
- Consistent **tagging** for roles and phases
- Friendly **fail-fast** checks (bad profile, bad phase override, missing phase task file)

---

## Structure

    _common/
    ├── tasks/
    │   └── task_flow.yml          # orchestrator (profile engine)
    └── vars/
        └── task_flow.yml          # canonical phase order + profiles

---

## Canonical phase order

The canonical phase order is defined once in:

- `_common/vars/task_flow.yml` → `task_flow.order`

Example:

    task_flow:
      order:
        - assert
        - install
        - config
        - validate

When you add a new phase (ex: `backup`), you add it **here** in the correct order.

---

## Profiles (categories)

Profiles define the phase subset each role runs. They live in:

- `_common/vars/task_flow.yml` → `task_flow.profiles`

Default profiles:

- `full`       → `[assert, install, config, validate]`
- `config`     → `[assert, config, validate]`
- `guard`      → `[assert]`
- `full_dev`   → `[assert, install, config]` (skip validate while iterating)
- `config_dev` → `[assert, config]` (skip validate while iterating)

---

## How roles use it

Each role calls the orchestrator from its `tasks/main.yml` and provides:

- `_role_tag` (for tagging)
- `_role_enabled` (enable/disable switch)
- `_task_flow_profile` (one of the profiles)

Example: **full** role

    ---
    - name: Execute common task flow
      ansible.builtin.include_tasks: "{{ role_path }}/../../_common/tasks/task_flow.yml"
      vars:
        _role_tag: "base_bootstrap"
        _role_enabled: "{{ base_bootstrap_enabled | default(true) | bool }}"
        _task_flow_profile: "full"

Example: **config-only** role

    ---
    - name: Execute common task flow
      ansible.builtin.include_tasks: "{{ role_path }}/../../_common/tasks/task_flow.yml"
      vars:
        _role_tag: "base_sshd"
        _role_enabled: "{{ base_sshd_enabled | default(true) | bool }}"
        _task_flow_profile: "config"

Example: **guard** role

    ---
    - name: Execute common task flow
      ansible.builtin.include_tasks: "{{ role_path }}/../../_common/tasks/task_flow.yml"
      vars:
        _role_tag: "base_preflight"
        _role_enabled: "{{ base_preflight_enabled | default(true) | bool }}"
        _task_flow_profile: "guard"

---

## Required per-role phase files

If a profile includes a phase, the role should provide:

- `roles/<role_name>/tasks/assert.yml`
- `roles/<role_name>/tasks/install.yml` (only if profile includes install)
- `roles/<role_name>/tasks/config.yml`
- `roles/<role_name>/tasks/validate.yml` (only if profile includes validate)

The orchestrator performs a friendly pre-check and fails fast if a required file is missing.

---

## Enable/disable roles

Each role should have a boolean in `defaults/main.yml`:

    base_bootstrap_enabled: true

Then override in inventory:

    # group_vars/lab.yml
    base_bootstrap_enabled: false

When disabled, the orchestrator exits the role immediately (`meta: end_role`).

---

## Tagging behavior

Every included phase task file receives tags:

- `<role_tag>` and `<phase>`

So you can run:

    # Run only this role (enabled roles only)
    ansible-playbook -i inventories/lab/hosts.yml playbooks/site.yml --tags base_bootstrap

    # Run install phase across all enabled roles in the play
    ansible-playbook -i inventories/lab/hosts.yml playbooks/site.yml --tags install

    # Run validate across all enabled roles in the play
    ansible-playbook -i inventories/lab/hosts.yml playbooks/site.yml --tags validate

    # Skip validate everywhere
    ansible-playbook -i inventories/lab/hosts.yml playbooks/site.yml --skip-tags validate

---

## Escape hatch: override phases (rare)

For exceptional cases only, you can bypass the profile and provide an explicit phase list:

    ---
    - name: Execute common task flow (override phases)
      ansible.builtin.include_tasks: "{{ role_path }}/../../_common/tasks/task_flow.yml"
      vars:
        _role_tag: "some_role"
        _role_enabled: "{{ some_role_enabled | default(true) | bool }}"
        _task_flow_profile: "config"   # ignored because override is set
        _task_flow_phases_override:
          - assert
          - install
          - config

The orchestrator validates override phases against the canonical order and fails fast on typos.

---

## Adding a new phase (example: `backup`)

1) Update `_common/vars/task_flow.yml`
- Add `backup` to `task_flow.order`
- Add `backup` to any profiles that should include it

2) Add `tasks/backup.yml` to roles that use profiles including `backup`

3) Use tags to run it:
- `--tags backup`
- `--tags <role_tag>,backup`

---

## Conventions (recommended)

- `assert`: validate inputs, OS checks, guardrails
- `install`: apt packages/repos, binaries, base users/groups if needed
- `config`: templates, sysctl, systemd units, enable/restart handlers
- `validate`: verify services, ports, config syntax, health checks
