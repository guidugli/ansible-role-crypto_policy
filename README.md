# Ansible Role: crypto_policy

[![CI](https://github.com/guidugli/ansible-role-crypto_policy/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-crypto_policy/actions/workflows/CI.yml)
[![Release](https://github.com/guidugli/ansible-role-crypto_policy/actions/workflows/release.yml/badge.svg)](https://github.com/guidugli/ansible-role-crypto_policy/actions/workflows/release.yml)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.crypto__policy-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/crypto_policy/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

Manage Fedora system crypto policy with a validation-first, idempotent Ansible role.

## Overview

This role installs the required Fedora crypto policy packages, validates the requested
policy against the target host's available policies and subpolicies, and applies the
requested policy with `update-crypto-policies`.

The role is intentionally scoped to Fedora only.

## Features

- Fedora-only support with explicit platform assertions.
- Automatic Ansible argument validation via `meta/argument_specs.yml`.
- Semantic validation in `tasks/assert.yml`.
- Idempotent policy application.
- Optional FIPS mode enablement, disabled by default.
- Generator-based Galaxy metadata refresh.
- Molecule coverage using a shared playbook structure.

## Supported platforms

- Fedora 42
- Fedora 43

## Role variables

Defaults are defined in [`defaults/main.yml`](defaults/main.yml).

```yaml
enable_crypto_policy: true
crypto_policy: DEFAULT
crypto_policies_reload: false
crypto_reboot_after_update: false
crypto_policy_manage_fips_mode: false
crypto_policy_packages:
  - crypto-policies
  - crypto-policies-scripts
```

### Important behavior

- `crypto_policy` accepts a base policy with optional subpolicies, for example
  `DEFAULT` or `DEFAULT:NO-SHA1`.
- The role validates the requested policy against the policy files available on the target.
- `crypto_policy_manage_fips_mode` is separate from `crypto_policy: FIPS` because enabling
  OS-level FIPS mode is a broader operational change than setting the crypto policy value.

## How it works

1. Validate public inputs with role argument specs.
2. Assert Fedora support and semantic input correctness.
3. Install required packages.
4. Discover available policies and current system state.
5. Validate the requested base policy and subpolicies against the target.
6. Apply the policy only when the current policy differs.
7. Optionally enable FIPS mode and reboot when requested.

## Usage

### Basic usage

```yaml
---
- name: Manage crypto policy
  hosts: fedora_hosts
  become: true
  roles:
    - role: guidugli.crypto_policy
```

### Custom policy example

```yaml
---
- name: Apply a specific Fedora crypto policy
  hosts: fedora_hosts
  become: true
  roles:
    - role: guidugli.crypto_policy
      vars:
        crypto_policy: DEFAULT:NO-SHA1
        crypto_policies_reload: false
```

### FIPS mode example

```yaml
---
- name: Enable FIPS policy and FIPS mode
  hosts: fedora_hosts
  become: true
  roles:
    - role: guidugli.crypto_policy
      vars:
        crypto_policy: FIPS
        crypto_policy_manage_fips_mode: true
        crypto_reboot_after_update: true
```

## Design notes

- The role does **not** force `become` in tasks. Callers should set `become: true` in the play.
- Reboot handling is delegated to a handler and skipped automatically in containerized Molecule runs.
- The role relies on standard gathered facts and should normally run with `gather_facts: true`.

## Molecule testing

The repository uses a shared Molecule playbook structure:

- `molecule/shared/vars.yml`
- `molecule/shared/converge.yml`
- `molecule/shared/verify.yml`
- `molecule/default/`

Run locally:

```bash
python -m venv .venv
. .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install "ansible-core>=2.16,<2.19" ansible-lint yamllint molecule "molecule-plugins[docker]"
ansible-galaxy collection install -r requirements.yml
./scripts/update_release_metadata.sh
molecule test
```

## Release workflow

- `scripts/update_release_metadata.sh` performs a fail-fast syntax check on generator scripts.
- `scripts/render_meta_main.py` renders `meta/main.yml` from `templates/meta_main.yml.j2`
  and `molecule/shared/vars.yml`.
- CI verifies that generated metadata is up to date.
- Tagged pushes trigger the release workflow.

## Repository structure

```text
.
├── defaults/
├── handlers/
├── meta/
├── molecule/
│   ├── default/
│   └── shared/
├── scripts/
├── tasks/
└── templates/
```

## License

MIT

## Author

Carlos Guidugli
