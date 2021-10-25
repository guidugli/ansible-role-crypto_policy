Ansible Role: crypto_policy
=========

An Ansible Role that install and configure crypto_policy on RHEL/CentOS, Fedora and Debian/Ubuntu.

Requirements
------------

No Requirements.

Role Variables
--------------

**Available variables are listed below, along with default values (see defaults/main.yml):**

    enable_crypto_policy: "{{ _enable_crypto_policy }}"

If true/yes, install and configure crypto policy on the system. Default settings will only configure cryto policy on RedHat/CentOS/Fedora systems.

    crypto_policy: DEFAULT

Valid values are DEFAULT, FUTURE and FIPS.
Recommended using DEFAULT or FUTURE.
LEGACY is not allowed for security reasons.

    crypto_reboot_after_update: yes

If policy is updated, reboot the target device?

**The variables listed below do not need to be changed for targeted systems (see vars/main.yml):**

    crypto_policy_packages:

Packages to install crypto_policy.

Dependencies
------------

No dependencies.

Example Playbook
----------------

    - hosts: servers
      roles:
         - { role: guidugli.crypto_policy }

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2020 by Carlos Guidugli.
