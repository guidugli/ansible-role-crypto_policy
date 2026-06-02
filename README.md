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

The role is intentionally scoped to Fedora execution, while keeping the shared
platform-matrix/generator pattern used across the rest of the role ecosystem.

## Features

- Fedora-only runtime support with explicit platform assertions.
- Automatic Ansible argument validation via `meta/argument_specs.yml`.
- Semantic validation in `tasks/assert.yml`.
- Idempotent crypto policy application.
- Optional FIPS mode enablement, disabled by default.
- Generator-based metadata and Molecule inventory refresh.
- Shared Molecule structure with `default` and `systemd` scenarios.
- CI, release, and local helper scripts aligned with the current repository pattern.

## Supported platforms

### Runtime support

- Fedora

### Molecule test coverage

- Fedora 44
- Fedora 43

> `meta/main.yml` is generated and currently renders Fedora as `all`, while Molecule
> inventories are generated from `molecule/shared/vars.yml` and currently test Fedora 44/43.

## Role variables

Defaults are defined in [`defaults/main.yml`](defaults/main.yml).

```yaml
---
enable_crypto_policy: true
crypto_policy: DEFAULT
crypto_policies_reload: false
crypto_reboot_after_update: false
crypto_policy_manage_fips_mode: false
crypto_policy_packages:
  - crypto-policies
  - crypto-policies-scripts
```

### Variables resolved from `vars/main.yml`

The repository keeps the shared pattern of deriving some internal values from
`vars/main.yml`. That file now uses `ansible_facts[...]` accessors to avoid
`INJECT_FACTS_AS_VARS` deprecation warnings while preserving the same generator-friendly
role structure used in your other roles.

## Important behavior

- `crypto_policy` accepts a base policy with optional subpolicies, for example
  `DEFAULT` or `DEFAULT:NO-SHA1`.
- The role validates the requested policy against the policy files available on the target.
- `crypto_policy_manage_fips_mode` is separate from `crypto_policy: FIPS` because enabling
  OS-level FIPS mode is a broader operational change than setting the crypto policy value.
- The role does not force `become` in role tasks. Callers should set privilege escalation
  at play level when needed.

## How it works

1. Ansible performs automatic role argument validation using `meta/argument_specs.yml`.
2. `tasks/assert.yml` enforces Fedora-only runtime support and semantic validation.
3. The role installs required packages.
4. The role discovers the current crypto policy plus available base policies and subpolicies.
5. The requested policy is normalized and validated.
6. The role applies the requested policy only when a change is required.
7. Optional FIPS enablement and reboot handling are triggered only when requested.

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
        crypto_policies_reload: true
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

- Runtime support is Fedora-only by design.
- The repository still uses the same generated/shared metadata and inventory flow used by your
  other modernized roles, even though this role only executes on Fedora.
- Reboot handling is delegated to a handler and skipped automatically when the environment is
  detected as a container.
- The role expects gathered facts to be available.

## Molecule testing

The repository uses a shared Molecule structure:

- `molecule/shared/vars.yml`
- `molecule/shared/converge.yml`
- `molecule/shared/verify.yml`
- `molecule/default/`
- `molecule/systemd/`

### Scenarios

- `default`: Podman-based container validation.
- `systemd`: Podman-based systemd-capable container validation.

### Local run

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
./scripts/update_release_metadata.sh
molecule test -s default
molecule test -s systemd
```

## Metadata / inventory generation

The repository follows the same template-first and generator-based pattern used in your other
modernized roles:

- `scripts/update_release_metadata.sh`
- `scripts/render_inventory.py`
- `scripts/render_meta_main.py`
- `templates/meta_main.yml.j2`
- `molecule/shared/vars.yml`

`update_release_metadata.sh` syntax-checks the generator scripts, optionally refreshes the shared
platform matrix, regenerates Molecule inventories, and renders `meta/main.yml`.

## Release workflow

- CI runs Molecule for the configured scenarios.
- Tagged pushes matching `v*` trigger the release workflow.
- `scripts/release.sh` can be used to refresh generated assets, run tests, create the release
  commit/tag, and push the result.

## Repository structure

```text
.
├── defaults/
├── handlers/
├── meta/
├── molecule/
│   ├── default/
│   ├── shared/
│   └── systemd/
├── scripts/
├── tasks/
├── templates/
├── tests/
└── vars/
```

## License

MIT

## Author

Carlos Guidugli
