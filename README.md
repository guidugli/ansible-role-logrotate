[![CI](https://github.com/guidugli/ansible-role-logrotate/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-logrotate/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/tag/guidugli/ansible-role-logrotate?sort=semver)](https://github.com/guidugli/ansible-role-logrotate/tags)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.logrotate-blue)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/logrotate/)
[![License](https://img.shields.io/github/license/guidugli/ansible-role-logrotate)](https://github.com/guidugli/ansible-role-logrotate/blob/main/LICENSE)

# Ansible Role: logrotate

Install and configure `logrotate`, keep global rotation defaults consistent, and optionally deploy reusable logrotate snippets.

This role currently manages:

- installation of the `logrotate` package;
- global retention via `rotate` in `/etc/logrotate.conf`;
- global `dateext` and `compress` directives;
- optional deployment of additional snippets into the logrotate include directory;
- conditional management of `logrotate.timer` when the target uses `systemd` and the timer unit is present.

## Requirements

- Ansible `2.14` or newer for the role metadata currently declared by the project.
- `containers.podman` version `>=1.10.0` for the Molecule scenarios used by this repository.
- Privileged execution on real hosts when installing packages or editing files under `/etc`.

## Variables

All role inputs are defined in `defaults/main.yml` and validated by `meta/argument_specs.yml`.

| Variable | Type | Default | Purpose |
| --- | --- | --- | --- |
| `logrotate_compress` | `bool` | `false` | Toggle the global `compress` directive in `/etc/logrotate.conf`. |
| `logrotate_use_date_extension` | `bool` | `true` | Toggle the global `dateext` directive in `/etc/logrotate.conf`. |
| `logrotate_weeks2keep` | `int` | `24` | Set the global `rotate` value used for retention. |
| `logrotate_config_files` | `list[str]` | `[]` | Copy additional OS-family-specific snippets from `files/<os_family>/` into `{{ logrotated_path }}`. |

### Internal role variables

The following variables are computed in `vars/main.yml` and normally do **not** need to be overridden:

- `logrotate_packages`
- `logrotate_conf`
- `logrotated_path`

## Example playbook

### Basic global configuration

```yaml
---
- name: Configure logrotate defaults
  hosts: all
  become: true
  roles:
    - role: guidugli.logrotate
      vars:
        logrotate_compress: true
        logrotate_use_date_extension: true
        logrotate_weeks2keep: 12
```

### Deploy additional snippets

```yaml
---
- name: Configure logrotate and deploy application snippets
  hosts: all
  become: true
  roles:
    - role: guidugli.logrotate
      vars:
        logrotate_config_files:
          - custom-app
          - custom-nginx
```

> Place the snippet files under `files/<os_family>/` using the target host OS family name, because the role copies them from `{{ ansible_facts['os_family'] }}`.

## Molecule testing

This repository contains two Molecule scenarios:

- `default`
- `systemd`

The shared platform matrix currently covers:

- Ubuntu `26.04`, `24.04`
- Debian `13`, `12`
- Fedora `44`, `43`

Run the full local test workflow with:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

### Verify-stage note

The repository includes `tasks/verify_context.yml` as an intentionally empty helper so Molecule verify can load role defaults before asserting outcomes.

## Execution notes

- The role itself does **not** set `become`; privilege must be controlled by the calling play.
- Package installation and writes to `/etc/logrotate.conf` and `{{ logrotated_path }}` require elevated privileges on real hosts.
- The Molecule shared converge play runs the role without forcing privilege escalation; test containers typically run as `root`.
- `logrotate.timer` is managed only when `ansible_facts['service_mgr'] == 'systemd'` and a timer unit file is found.
- Non-`systemd` containers remain supported because the timer task is conditionally guarded.
- The `systemd` Molecule scenario exists specifically to exercise the timer-management path in a containerized environment.

## Maintenance notes

- `templates/meta_main.yml.j2` is the generator input for role metadata; generated outputs should be refreshed through the repository scripts rather than edited manually.
- `meta/main.yml` is generated content and should stay aligned with the template and shared Molecule matrix.
