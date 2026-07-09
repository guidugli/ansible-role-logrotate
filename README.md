[![CI](https://github.com/guidugli/ansible-role-logrotate/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-logrotate/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/tag/guidugli/ansible-role-logrotate?sort=semver)](https://github.com/guidugli/ansible-role-logrotate/tags)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.logrotate-blue)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/logrotate/)
[![License](https://img.shields.io/github/license/guidugli/ansible-role-logrotate)](https://github.com/guidugli/ansible-role-logrotate/blob/main/LICENSE)

# Ansible Role: logrotate

Install and configure logrotate, keep global rotation defaults consistent, and optionally deploy reusable logrotate snippets.

## Requirements

- Ansible 2.14 or newer for role metadata compatibility.
- `containers.podman` collection version `>=1.10.0` for Molecule scenarios.

## Variables

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `logrotate_compress` | `bool` | `false` | Enable compression for rotated logs in the global configuration. |
| `logrotate_use_date_extension` | `bool` | `true` | Use date-based suffixes instead of numeric suffixes. |
| `logrotate_weeks2keep` | `int` | `24` | Set the global `rotate` retention value. |
| `logrotate_config_files` | `list[str]` | `[]` | Copy OS-family-specific snippets from `files/<os_family>/` into `{{ logrotated_path }}`. |

## Example Playbook

```yaml
---
- name: Configure logrotate
  hosts: all
  become: true
  roles:
    - role: guidugli.logrotate
      vars:
        logrotate_compress: true
        logrotate_use_date_extension: true
        logrotate_weeks2keep: 12
        logrotate_config_files:
          - custom-app
```

## Molecule Testing

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

## Execution Notes

- The role does not set `become`; privilege must be controlled by the caller.
- Package installation, edits under `/etc`, and optional snippet deployment require elevated privileges on real hosts.
- Default Molecule containers typically execute as root, so the shared converge play does not force privilege escalation.
- `logrotate.timer` is managed only when the target uses `systemd` and the timer unit is present.
- Non-systemd containers remain supported because the service task is guarded on `ansible_facts['service_mgr']`.
