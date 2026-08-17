[![CI](https://github.com/guidugli/ansible-role-logrotate/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-logrotate/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/tag/guidugli/ansible-role-logrotate?sort=semver)](https://github.com/guidugli/ansible-role-logrotate/tags)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.logrotate-blue)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/logrotate/)
[![License](https://img.shields.io/github/license/guidugli/ansible-role-logrotate)](https://github.com/guidugli/ansible-role-logrotate/blob/main/LICENSE)

# Ansible Role: logrotate

Install and configure `logrotate`, keep global rotation defaults consistent, and optionally deploy reusable logrotate snippets. The role is template-aligned, generator-safe, Molecule-ready, and execution-context aware for real hosts and containers.

## Requirements

- Ansible `2.14` or newer, matching the role metadata declared by this repository.
- `containers.podman` collection `>=1.10.0` for the Molecule scenarios.
- Privileged execution from the calling play is required on real hosts for package installation and writes under `/etc`.
- The role does not install or configure cron; it relies on the platform-provided logrotate package behavior and conditionally manages `logrotate.timer` on systemd hosts.

## Features

- Installs the platform `logrotate` package.
- Sets global rotation retention in `/etc/logrotate.conf`.
- Controls the global `dateext` directive.
- Controls the global `compress` directive.
- Copies optional OS-family-specific snippets into the logrotate include directory.
- Starts and enables `logrotate.timer` only when the target service manager is systemd and a timer unit exists.
- Uses fully qualified Ansible module names and `ansible_facts[...]` access patterns.
- Leaves privilege escalation to the caller.

## Supported platforms

The generated metadata and Molecule matrix are driven from `molecule/shared/vars.yml` and `templates/meta_main.yml.j2`.

| Family | Versions currently represented in the shared matrix |
| --- | --- |
| Ubuntu | `26.04`, `24.04` |
| Debian | `13`, `12` |
| Fedora | `44`, `43` |

## Variables

All user inputs are defined in `defaults/main.yml` and validated in `meta/argument_specs.yml`.

| Variable | Type | Default | Required | Description |
| --- | --- | --- | --- | --- |
| `logrotate_compress` | `bool` | `false` | No | When `true`, ensures the global `compress` directive is present in `/etc/logrotate.conf`. When `false`, the role writes `#compress` for the global directive. |
| `logrotate_use_date_extension` | `bool` | `true` | No | When `true`, ensures the global `dateext` directive is present in `/etc/logrotate.conf`. When `false`, the role writes `#dateext` for the global directive. |
| `logrotate_weeks2keep` | `int` | `24` | No | Sets the global `rotate` value in `/etc/logrotate.conf`. The value must be greater than zero. |
| `logrotate_config_files` | `list[str]` | `[]` | No | Optional list of snippet file names to copy from `files/{{ ansible_facts['os_family'] }}/` into `{{ logrotated_path }}`. Use file names only, not full paths. |

Internal variables are computed in `vars/main.yml` and normally do not need to be overridden:

| Variable | Purpose |
| --- | --- |
| `logrotate_packages` | Package list installed by the role. Defaults to `['logrotate']`. |
| `logrotate_conf` | Global logrotate configuration file path. Defaults to `/etc/logrotate.conf`. |
| `logrotated_path` | Directory for snippet configuration files. Defaults to `/etc/logrotate.d`. |

## Example playbook

Use privilege escalation in the calling play when the target host requires it.

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

To deploy additional snippets, place files below `files/<os_family>/` and list only the snippet names:

```yaml
---
- name: Configure logrotate snippets
  hosts: all
  become: true
  roles:
    - role: guidugli.logrotate
      vars:
        logrotate_config_files:
          - custom-app
          - custom-nginx
```

## Molecule testing instructions

The repository provides two Molecule scenarios:

- `default`, for standard container execution.
- `systemd`, for the timer-management path in systemd-capable containers.

Run the local validation flow with:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

The shared verify play checks that the package is installed and that the configured retention value is present in `/etc/logrotate.conf`.

## Execution notes

- The role never sets `become`, `become_user`, or `become_method` in executable role tasks.
- Package installation and changes to `/etc/logrotate.conf` and `/etc/logrotate.d` are privileged operations on real hosts.
- Molecule containers run the role without forcing privilege escalation because the test containers are expected to run as root.
- Systemd handling is guarded by `ansible_facts['service_mgr'] == 'systemd'` and by a pre-check for a discovered `logrotate.timer` unit file.
- Non-systemd containers skip the timer-management task and still exercise package and configuration behavior.
- If target hosts require privilege, set `become: true` in the calling play, inventory, or automation controller job template.

## Release workflow

Release metadata is generator-driven:

- Edit `templates/meta_main.yml.j2` for role metadata inputs.
- Keep platform data in `molecule/shared/vars.yml`.
- Refresh generated metadata and inventories with `scripts/update_release_metadata.sh`.
- Validate with `yamllint`, `ansible-lint`, and both Molecule scenarios.
- Create a tagged release with `scripts/release.sh --version vX.Y.Z` when the repository is clean.

Generated files such as `meta/main.yml` and scenario inventories should be produced by the repository scripts rather than edited directly.

## License

MIT. See `LICENSE` for the full license text.

## Author

Carlos Guidugli.
