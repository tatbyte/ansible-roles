# _common

Shared “framework” files used by all roles.

This folder provides a **task flow orchestrator + quiet wrapper** that standardizes how roles run ordered phases (assert → install → config → validate), with:

- **Profiles** (full/config/guard + dev variants)
- Per-role **enable/disable** switch
- Consistent **tagging** for roles and phases
- Friendly **fail-fast** checks (bad profile, bad phase override, missing phase task file)
- Optional **debug/summary** output with minimal noise in normal runs

---

## Structure

    _common/
    ├── defaults/
    │   └── main.yml               # summary/debug defaults, wrapper success toggle
    ├── tasks/
    │   ├── main.yml               # wrapper + fail handling
    │   └── task_flow.yml          # orchestrator + role/phase summaries
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
- `guard`      → `[assert, validate]`
- `full_dev`   → `[assert, install, config]` (skip validate while iterating)
- `config_dev` → `[assert, config]` (skip validate while iterating)

---

## How roles use it

Roles should include the **quiet wrapper** `../_common/tasks/main.yml` from their `tasks/main.yml` and provide:

- `_role_tag` (for tagging/summary label)
- `_role_enabled` (enable/disable switch)
- `_task_flow_profile` (one of the profiles)
- Optional inventory knobs:
  - `task_flow_summary_mode` (`phase|role|none`)
  - `task_flow_debug` (`true|false`)
  - `task_flow_role_success_output` (`true|false`, controls wrapper `_common passed` line)

Example: **full** role

    ---
    - name: Execute common task flow
      ansible.builtin.include_tasks: "{{ role_path }}/../_common/tasks/main.yml"
      vars:
        _role_tag: "base_bootstrap"
        _role_enabled: "{{ base_bootstrap_enabled | default(true) | bool }}"
        _task_flow_profile: "full"

Example: **config-only** role

    ---
    - name: Execute common task flow
      ansible.builtin.include_tasks: "{{ role_path }}/../_common/tasks/main.yml"
      vars:
        _role_tag: "base_sshd"
        _role_enabled: "{{ base_sshd_enabled | default(true) | bool }}"
        _task_flow_profile: "config"

Example: **guard** role

    ---
    - name: Execute common task flow
      ansible.builtin.include_tasks: "{{ role_path }}/../_common/tasks/main.yml"
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

Wrapper summary/debug knobs (inventory-level):

- `task_flow_summary_mode`:
  - `phase`: ✅ per phase
  - `role`: single ✅ per role (`[OK] <role_tag> :: role passed`)
  - `none`: no success lines (failures still show)
- `task_flow_debug`: `true|false` to print selection + phase start breadcrumbs
- `task_flow_role_success_output`:
  - `false`: suppress wrapper line `[OK] <role_tag> :: _common passed` (default)
  - `true`: print wrapper line when `task_flow_summary_mode == role`

### Where summary lines come from

- `task_flow.yml` emits:
  - phase summary (`[OK] <role_tag> :: <phase> passed`) when `task_flow_summary_mode == phase`
  - role summary (`[OK] <role_tag> :: role passed`) when `task_flow_summary_mode == role`
- `main.yml` wrapper can emit:
  - `[OK] <role_tag> :: _common passed` when `task_flow_role_success_output == true`

This split lets consuming repos keep `_common` internal usage while showing only role-level summaries.

### Recommended quiet output setup (consuming repo)

In the consuming project (`ansible.cfg`):

    [defaults]
    callback_plugins = ./callback_plugins
    stdout_callback = role_summary

And in `_common/defaults/main.yml` (or inventory):

    task_flow_summary_mode: "role"
    task_flow_role_success_output: false
    task_flow_debug: false

With this setup:

- Roles still run through `_common`
- Internal `_common` task noise is suppressed by callback output filtering
- You keep one line per role: `[OK] <role_tag> :: role passed`
- Failures/unreachable still surface clearly

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
      ansible.builtin.include_tasks: "{{ role_path }}/../_common/tasks/main.yml"
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
- Set `task_flow_debug: true` temporarily when diagnosing profile/override issues; use `task_flow_summary_mode: phase|role|none` to control success chatter.
