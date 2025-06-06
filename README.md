# Users
Manage users and groups in a sane and programmatic way for easy role and user
consumption.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_users/blob/main/meta/main.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_users/tree/main/defaults/main/)

## Dependencies
**galaxy-ng** roles cannot be used independently. Part of
[r_pufky.deb](https://github.com/r-pufky/ansible_collection_deb) collection.

## Example Playbook
Consume this role to easily create consistent complex users and groups across a
fleet of machines and roles.

### User/Group Definitions
Host inventory is parsed for user/group definitions:

* `users_group_{LABEL}` defines a group.
* `users_user_{LABEL}` defines a user.

And applied based on `users_accounts_*` variables.

Labels are generally usernames but may be used to specify multiple variants of
the same account (e.g. `users_user_example` and `users_user_example_no_ssh`).

Definitions are generally placed in `group_vars` but may also be specified
per-host to explicitly define these per system.

group_vars/all/users.yml
``` yaml
users_group_root:
  name: 'root'
  gid: 0
users_user_root:
  name: 'root'
  group: 'root'
  uid: 0
  extensions:
    include_group: true
users_group_ansible:
  name: 'ansible'
  gid: 900
users_user_ansible:
  name: 'ansible'
  group: 'ansible'
  uid: 900
  extensions:
    include_group: true
users_group_example:
  name: 'example'
  gid: 1000
users_user_example:
  name: 'example'
  comment: 'Interactive user'
  home: '/home/example'
  group: 'example'
  uid: 1000
  extensions:
    include_group: true
    sudoers:
      commands: 'ALL'
      group: 'sudo'
      host: 'ALL'
      name: 'example_user_sudo'
      noexec: true
      nopassword: true
      runas: 'root'
      setenv: true
      validation: 'detect'
    ssh:
      ssh_keys:
        - name: 'id_ed25519'
          public: '{{ vault_example_ssh_public_key }}'
          private: '{{ vault_example_ssh_private_key }}'
      authorized_keys:
        - key: 'ssh-rsa AAAA...9Rm public_key@example'
      config: |
        # auto-accept new host keys
        Host *
          StrictHostKeyChecking accept-new
    gpg:
      config: |
        personal-cipher-preferences AES256 AES192 AES
        personal-digest-preferences SHA512 SHA384 SHA256
        personal-compress-preferences ZLIB BZIP2 ZIP Uncompressed
      gpg_keys:
        - note: 'Example keys from https://www.ietf.org/id/draft-bre-openpgp-samples-01.html'
          fingerprint: 'EB85BB5FA33A75E15E944E63F231550C4F47E38E'
          public_key: '{{ vault_example_gpg_public_key }}'
          private_key: '{{ vault_example_gpg_private_key }}'
          trust: 5
    copy_data: 'group_vars/data/users/example'
    lockdown: true
users_user_example_no_extensions:
  name: 'example'
  comment: 'Interactive user'
  home: '/home/example'
  group: 'example'
  uid: 1000
```

### Managing Users/Groups
Role will apply users/groups based on finding the user/group definition
variables and applying those. These are generally applied per machine in
`host_vars`.

host_vars/client.example.com/vars/users.yml
``` yaml
users_accounts_users:
  - 'root'
  - 'ansible'
users_accounts_users_absent:
  - 'example'
```

host_vars/webserver.example.com/vars/users.yml
``` yaml
users_accounts_users:
  - 'ansible'
  - 'example_no_extensions'
```

site.yml
``` yaml
- name: 'Manage system accounts'
  hosts: 'all'
  tasks:
  - name: 'manage users and groups'
    ansible.builtin.include_role:
      name: 'r_pufky.deb.users'
```

### Explicit Definitions
Standard role playbook practices apply; you may define everything when
including the role.

playbook.yml
``` yaml
- name: 'Manage system accounts'
  hosts: 'all'
  tasks:
  - name: 'manage users and groups'
    ansible.builtin.include_role:
      name: 'r_pufky.deb.users'
    vars:
      users_user_root:
        name: 'root'
        uid: 0
      users_user_example:
        name: 'example'
        uid: 1600
      users_accounts_users:
        - 'root'
      users_accounts_users_absent:
        - 'example'
```

### Role Playbook
Roles many consume this role to easily create mandatory users and groups
required for execution. Run once for each user/group to create.

Default user/group settings are focused on service deployments and are
different then standard accounts. It is always best to explicitly set options.

roles/my_custom_role/tasks/add_mandatory_users.yml
``` yaml
- name: 'add required service user/groups'
  ansible.builtin.include_role:
    name: 'r_pufky.deb.users'
    tasks_from: 'role_account_add'
  vars:
    users_role_user:
      name: 'my_service_user'
      groups: 'my_service_group'
      uid: 800
      system: true
    users_role_group:
      name: 'my_service_group'
      gid: 800
```

### Debian Bookworm Changes

#### ssh group now _ssh
ssh group migrated to **_ssh**. ssh group must be manually managed if used with
users and groups, or migrate users to _ssh.

https://salsa.debian.org/ssh-team/openssh/-/commit/18da782ebe789d0cf107a550e474ba6352e68911

#### SSH pubkey authentication with locked passwords
Locked accounts cannot SSH pubkey auth. SSH now distinguishes between `!` and
`*` password locking:
  *: lock password, allow SSH pubkey auth
  !: lock password, deny SSH pubkey auth

Any other means to lock the password will result in SSH pubkey failures. Do not
enable `UsePam=yes` as this leads to security vulnerabilities.

https://unix.stackexchange.com/questions/193066/how-to-unlock-account-for-public-key-ssh-authorization-but-not-for-password-aut

## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_docs/blob/main/dev/environment/README.md)

Run all unit tests:
``` bash
molecule test --all
```

### Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_users/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)

