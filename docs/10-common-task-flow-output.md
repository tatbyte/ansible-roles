# _common Task Flow and Output Model

This document explains how `_common` runs role phases and how success/failure
output is controlled.

## Flow Overview

Roles include `_common/tasks/main.yml`, which calls `_common/tasks/task_flow.yml`.

High-level sequence:

1. load canonical phase configuration from `_common/vars/task_flow.yml`
2. compute role enabled state
3. validate profile or override phases
4. compute effective phase list
5. pre-check required phase task files
6. include role phase tasks in canonical order
7. emit summary lines based on settings

## Control Variables

Defaults are in `_common/defaults/main.yml`.

- `task_flow_summary_mode`
  - `phase`: print one success line per phase
  - `role`: print one success line per role
  - `none`: no success lines
- `task_flow_debug`
  - `true`: print debug breadcrumbs
  - `false`: quiet mode
- `task_flow_role_success_output`
  - `false` (default): suppress wrapper `_common passed` line
  - `true`: allow wrapper success line when summary mode is `role`

## Summary Line Sources

- From `_common/tasks/task_flow.yml`:
  - `[OK] <role_tag> :: <phase> passed` (`phase` mode)
  - `[OK] <role_tag> :: role passed` (`role` mode)
- From `_common/tasks/main.yml` wrapper:
  - `[OK] <role_tag> :: _common passed` (only when `task_flow_role_success_output: true`)

## Recommended Quiet Pattern (Consuming Repo)

In consuming repo `ansible.cfg`:

```ini
[defaults]
callback_plugins = ./callback_plugins
stdout_callback = role_summary
```

In inventory or defaults:

```yaml
task_flow_summary_mode: role
task_flow_debug: false
task_flow_role_success_output: false
```

Result:

- role uses `_common` orchestration
- internal `_common` noise is filtered by callback output mode
- one clean success line per role remains
- failures/unreachable remain visible
