# [Ansible role bareos_dir](#bareos_dir)

Install and configure [Bareos](https://www.bareos.com/) Director.

|GitHub|GitLab|Downloads|Version|Issues|Pull Requests|
|------|------|-------|-------|------|-------------|
|[![github](https://github.com/buluma/ansible-role-bareos_dir/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-bareos_dir/actions/workflows/molecule.yml)|[![gitlab](https://gitlab.com/shadowwalker/ansible-role-bareos_dir/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-bareos_dir)|[![downloads](https://img.shields.io/ansible/role/d/37530)](https://galaxy.ansible.com/buluma/bareos_dir)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-bareos_dir.svg)](https://github.com/buluma/ansible-role-bareos_dir/releases/)|[![Issues](https://img.shields.io/github/issues/buluma/ansible-role-bareos_dir.svg)](https://github.com/buluma/ansible-role-bareos_dir/issues/)|[![PullRequests](https://img.shields.io/github/issues-pr-closed-raw/buluma/ansible-role-bareos_dir.svg)](https://github.com/buluma/ansible-role-bareos_dir/pulls/)|

## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/buluma/ansible-role-bareos_dir/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: yes
  gather_facts: yes

  roles:
    - role: buluma.bareos_dir
      bareos_dir_backup_configurations: yes
      bareos_dir_install_debug_packages: yes
      bareos_dir_catalogs:
        - name: MyCatalog
          dbname: bareos
          dbuser: bareos
          dbpassword: ""
      bareos_dir_consoles:
        - name: bareos-mon
          description: "Restricted console used by tray-monitor to get the status of the director."
          password: "MySecretPassword"
          commandacl:
            - status
            - .status
          jobacl:
            - "*all"
      bareos_dir_clients:
        - name: bareos-fd
          address: 127.0.0.1
          password: "MySecretPassword"
          maximum_concurrent_jobs: 3
        - name: "disabled-client"
          enabled: no
      bareos_dir_filesets:
        - name: LinuxAll
          description: "Backup all regular filesystems, determined by filesystem type."
          include:
            files:
              - /
            exclude_dirs_containing: nobackup
            options:
              signature: MD5
              one_fs: no
              fs_types:
                - btrfs
                - ext2
                - ext3
                - ext4
                - reiserfs
                - jfs
                - vfat
                - xfs
                - zfs
              compression: GZIP
          exclude:
            files:
              - /var/lib/bareos
              - /var/lib/bareos/storage
              - /proc
              - /tmp
              - /var/tmp
              - /.journal
              - /.fsck
        - name: disabled-fileset
          enabled: no
      bareos_dir_jobdefs:
        - name: DefaultJob-1
          type: Backup
          level: Incremental
          fileset: SelfTest
          schedule: WeeklyCycle
          storage: File-1
          messages: Standard
          pool: Full
          priority: 10
          write_bootstrap: "/var/lib/bareos/%c.bsr"
          full_backup_pool: Full
          differential_backup_pool: Differential
          incremental_backup_pool: Incremental
        - name: "disabled-jobdef"
          enabled: no
      bareos_dir_jobs:
        - name: my_job
          description: "My backup job"
          pool: Full
          type: Backup
          client: bareos-fd
          fileset: LinuxAll
          storage: File-1
          messages: Standard
        - name: disabled_job
          enabled: no
        - name: BackupCatalog
          description: "Backup the catalog database (after the nightly save)"
          jobdefs: DefaultJob
          level: Full
          fileset: Catalog
          client: bareos-fd
          schedule: WeeklyCycleAfterBackup
          runbeforejob: "/usr/lib/bareos/scripts/make_catalog_backup MyCatalog"
          runafterjob: "/usr/lib/bareos/scripts/delete_catalog_backup MyCatalog"
          write_bootstrap: '|/usr/bin/bsmtp -h localhost -f \"\(Bareos\) \" -s \"Bootstrap for Job %j\" root'
          priority: 11
          maximum_concurrent_jobs: 2
      bareos_dir_messages:
        - name: "Standard"
          description: "Send relevant messages to the Director."
          append:
            - file: "/var/log/bareos/bareos.log"
              messages:
                - all
                - "!skipped"
                - "!terminate"
          catalog:
            - all
            - "!skipped"
            - "!saved"
            - "!audit"
          console:
            - all
            - "!skipped"
            - "!saved"
        - name: "disabled-message"
          enabled: no
        - name: Daemon
          description: "Message delivery for daemon messages (no job)."
          mailcommand: '/usr/bin/bsmtp -h localhost -f \"\(Bareos\) \<%r\>\" -s \"Bareos daemon message\" %r'
          mail:
            - to: root
              messages:
                - all
                - "!skipped"
                - "!audit"
          console:
            - all
            - "!skipped"
            - "!saved"
            - "!audit"
          append:
            - file: "/var/log/bareos/bareos.log"
              messages:
                - all
                - "!skipped"
                - "!audit"
            - file: "/var/log/bareos/bareos-audit.log"
              messages:
                - audit
        - name: RestoreFiles
          description: "Standard Restore template. Only one such job is needed for all standard Jobs/Clients/Storage ..."
          type: Restore
          client: bareos-fd
          fileset: LinuxAll
          storage: File-1
          pool: Incremental
          messages: Standard
          where: "/tmp/bareos-restores"
      bareos_dir_pools:
        - name: Full
          pool_type: Backup
          recycle: yes
          autoprune: yes
          volume_retention: 365 days
          maximum_volume_bytes: 50G
          maximum_volumes: 100
          label_format: "Full-"
        - name: "disabled-pool"
          enabled: no
      bareos_dir_profiles:
        - name: webui-admin
          jobacl:
            - "*all*"
          clientacl:
            - "*all*"
          storageacl:
            - "*all*"
          scheduleacl:
            - "*all*"
          poolacl:
            - "*all*"
          commandacl:
            - "!.bvfs_clear_cache"
            - "!.exit"
            - "!.sql"
            - "!configure"
            - "!create"
            - "!delete"
            - "!purge"
            - "!prune"
            - "!sqlquery"
            - "!umount"
            - "!unmount"
            - "*all*"
          filesetacl:
            - "*all*"
          catalogacl:
            - "*all*"
          whereacl:
            - "*all*"
          pluginoptionsacl:
            - "*all*"
        - name: "disabled-message"
          enabled: no
      bareos_dir_schedules:
        - name: WeeklyCycle
          run:
            - Full 1st sat at 21:00
            - Differential 2nd-5th sat at 21:00
            - Incremental mon-fri at 21:00
        - name: WeeklyCycleAfterBackup
          description: This schedule does the catalog. It starts after the WeeklyCycle.
          run:
            - Full mon-fri at 21:10
        - name: "disabled-schedule"
          enabled: no
      bareos_dir_storages:
        - name: File-1
          address: dir-1
          password: "MySecretPassword"
          device: FileStorage
          media_type: File
          tls_enable: yes
          tls_verify_peer: no
          maximum_concurrent_jobs: 3
        - name: "disabled-storage"
          enabled: no
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/buluma/ansible-role-bareos_dir/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: yes
  gather_facts: no

  roles:
    - role: buluma.bootstrap
    # The roles buildtools, python_pip and postgres are required.
    # bareos-dir needs to connect to a database.
    - role: buluma.buildtools
    # EPEL is required for RHEL7.
    - role: buluma.epel
    - role: buluma.python_pip
    - role: buluma.postgres
    # The roles core_dependencies and postfix are  required for the `bareos_role`: "dir".
    # bareos-dir needs to send emails.
    # - role: buluma.core_dependencies
    # - role: buluma.postfix
    - role: buluma.bareos_repository
      bareos_repository_enable_tracebacks: yes
```

Also see a [full explanation and example](https://buluma.github.io/how-to-use-these-roles.html) on how to use these roles.

## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/buluma/ansible-role-bareos_dir/blob/master/defaults/main.yml):

```yaml
---
# defaults file for bareos_dir

# The director has these configuration parameters.

# Backup the configuration files.
bareos_dir_backup_configurations: no

# Install debug packages. This requires the debug repositories to be enabled.
bareos_dir_install_debug_packages: no

# The hostname of the Director.
bareos_dir_hostname: "{{ inventory_hostname }}"

# The password for the Director.
bareos_dir_password: "secretpassword"

# The query file.
bareos_dir_queryfile: "/usr/lib/bareos/scripts/query.sql"

# The maximum number of concurrent jobs.
bareos_dir_max_concurrent_jobs: 100

# The messages configuration to use.
bareos_dir_message: Daemon

# Enable TLS.
bareos_dir_tls_enable: yes

# Verify the peer.
bareos_dir_tls_verify_peer: no

# A list of catalogs to configure.
bareos_dir_catalogs: []

# A list of consoled to configure.
bareos_dir_consoles: []

# A list of clients to configure.
bareos_dir_clients: []

# A list of filesets to configure.
bareos_dir_filesets: []

# A list of jobdefs to configure
bareos_dir_jobdefs: []

# A list of jobs to configure.
bareos_dir_jobs: []

# A list of messages to configure.
bareos_dir_messages: []

# A list of pools to configure.
bareos_dir_pools: []

# A list of profiles to configure.
bareos_dir_profiles: []

# A list of schedules to configure.
bareos_dir_schedules: []

# A list of storages to configure.
bareos_dir_storages: []
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/buluma/ansible-role-bareos_dir/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | GitLab |
|-------------|--------|--------|
|[buluma.bootstrap](https://galaxy.ansible.com/buluma/bootstrap)|[![Build Status GitHub](https://github.com/buluma/ansible-role-bootstrap/workflows/Ansible%20Molecule/badge.svg)](https://github.com/buluma/ansible-role-bootstrap/actions)|[![Build Status GitLab](https://gitlab.com/shadowwalker/ansible-role-bootstrap/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-bootstrap)|
|[buluma.bareos_repository](https://galaxy.ansible.com/buluma/bareos_repository)|[![Build Status GitHub](https://github.com/buluma/ansible-role-bareos_repository/workflows/Ansible%20Molecule/badge.svg)](https://github.com/buluma/ansible-role-bareos_repository/actions)|[![Build Status GitLab](https://gitlab.com/shadowwalker/ansible-role-bareos_repository/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-bareos_repository)|
|[buluma.buildtools](https://galaxy.ansible.com/buluma/buildtools)|[![Build Status GitHub](https://github.com/buluma/ansible-role-buildtools/workflows/Ansible%20Molecule/badge.svg)](https://github.com/buluma/ansible-role-buildtools/actions)|[![Build Status GitLab](https://gitlab.com/shadowwalker/ansible-role-buildtools/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-buildtools)|
|[buluma.epel](https://galaxy.ansible.com/buluma/epel)|[![Build Status GitHub](https://github.com/buluma/ansible-role-epel/workflows/Ansible%20Molecule/badge.svg)](https://github.com/buluma/ansible-role-epel/actions)|[![Build Status GitLab](https://gitlab.com/shadowwalker/ansible-role-epel/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-epel)|
|[buluma.python_pip](https://galaxy.ansible.com/buluma/python_pip)|[![Build Status GitHub](https://github.com/buluma/ansible-role-python_pip/workflows/Ansible%20Molecule/badge.svg)](https://github.com/buluma/ansible-role-python_pip/actions)|[![Build Status GitLab](https://gitlab.com/shadowwalker/ansible-role-python_pip/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-python_pip)|
|[buluma.postgres](https://galaxy.ansible.com/buluma/postgres)|[![Build Status GitHub](https://github.com/buluma/ansible-role-postgres/workflows/Ansible%20Molecule/badge.svg)](https://github.com/buluma/ansible-role-postgres/actions)|[![Build Status GitLab](https://gitlab.com/shadowwalker/ansible-role-postgres/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-postgres)|

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://buluma.github.io/) for further information.

Here is an overview of related roles:

![dependencies](https://raw.githubusercontent.com/buluma/ansible-role-bareos_dir/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/buluma):

|container|tags|
|---------|----|
|[Debian](https://hub.docker.com/repository/docker/buluma/debian/general)|bookworm, bullseye, buster|
|[EL](https://hub.docker.com/repository/docker/buluma/enterpriselinux/general)|7, 8, 9|
|[Fedora](https://hub.docker.com/repository/docker/buluma/fedora/general)|38|
|[opensuse](https://hub.docker.com/repository/docker/buluma/opensuse/general)|all|
|[Ubuntu](https://hub.docker.com/repository/docker/buluma/ubuntu/general)|jammy, focal|

The minimum version of Ansible required is 2.12, tests have been done to:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/buluma/ansible-role-bareos_dir/issues)

## [Changelog](#changelog)

[Role History](https://github.com/buluma/ansible-role-bareos_dir/blob/master/CHANGELOG.md)

## [License](#license)

[Apache-2.0](https://github.com/buluma/ansible-role-bareos_dir/blob/master/LICENSE).

## [Author Information](#author-information)

[buluma](https://buluma.github.io/)


### [Special Thanks](#special-thanks)

Template inspired by [Robert de Bock](https://github.com/robertdebock)
