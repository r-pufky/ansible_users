# Users
Manage users and groups in a sane and programmatic way for easy role and user
consumption.

## [Requirements][i]
Requires [r_pufky.deb][g] galaxy-ng collection.

## Role Variables
Detailed variable use documented in defaults. See usage for role operation.

* [defaults][j] - User configurable options.

* [account definitions][k] - Encapsulated user data structure.

## Usage
Consume this role to easily create consistent complex users and groups across a
fleet of machines and roles.

### Feature Flags
Tasks are gated by feature flags and executed in the following order.

  Step | Flag                   | Notes
 ------|------------------------|-------
  1    | users_flg_high_entropy | Install high-entropy packages.

### User/Group Definitions
Host inventory is parsed for user/group definitions:

* `users_group_{LABEL}` defines a group.
* `users_user_{LABEL}` defines a user.

And applied based on `users_cfg_*` variables.

Labels are generally usernames but may be used to specify multiple variants of
the same account (e.g. `users_user_example` and `users_user_example_no_ssh`).

Definitions are generally placed in `group_vars` but may also be specified
per-host to explicitly define these per system.

### Example Playbooks

#### Managing fleet
Role will apply users/groups based on finding the user/group definition
variables and applying those.

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

``` yaml
- name: 'Set users and groups for all hosts.'
  hosts: 'all'
  roles:
     - 'r_pufky.deb.users'
```

#### Managing per machine
Use `host_vars` to apply per machine settings.

host_vars/client.example.com/vars/users.yml
``` yaml
users_cfg_users:
  - 'root'
  - 'ansible'
users_cfg_users_absent:
  - 'example'
```

host_vars/webserver.example.com/vars/users.yml
``` yaml
users_cfg_users:
  - 'ansible'
  - 'example_no_extensions'
```

``` yaml
- name: 'Manage system accounts'
  hosts: 'all'
  tasks:
  - name: 'manage users and groups'
    ansible.builtin.include_role:
      name: 'r_pufky.deb.users'
```

#### Explicit Definitions
Standard role playbook practices apply; you may define everything when
including the role.

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
      users_cfg_users:
        - 'root'
      users_cfg_users_absent:
        - 'example'
```

## Debian 12+ Changes

### ssh group now _ssh
[ssh group migrated to **_ssh**](https://salsa.debian.org/ssh-team/openssh/-/commit/18da782ebe789d0cf107a550e474ba6352e68911).
ssh group must be manually managed if used with users and groups, or migrate
users to **_ssh**.

### SSH pubkey authentication with locked passwords
SSH now distinguishes between `!` and `*` password locking:

* *: lock password, allow SSH pubkey auth.
* !: lock password, deny SSH pubkey auth.

Any other means to lock the password will result in SSH pubkey failures. Do
NOT set ssh_cfg_server_use_pam=true as this leads to
[security vulnerabilities][l] with potential [mitigataions][m] if absolutely
necessary.

## Development
Configure [environment][a].

``` bash
# Run all tests.
molecule test --all
```

### [Releases][b]

  Release | Debian | Ansible | Notes
 ---------|--------|---------|-------
  3.x.x   | 13     | 2.20    | Ansible 2.20, semantic versioning.
  2.x.x   | 13     | 2.18    | Migrate to Debian Trixie.
  1.x.x   | 12     | 2.18    | Use standardized libraries.
  0.x.x   | 12     | 2.18    | Migration from private repository.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License][c] | [direct link][f]

## Author Information
PGP: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9][d] | [github gist][e]


[a]: https://r-pufky.github.io/ansible_docs
[b]: https://semver.org/spec/v2.0.0
[c]: https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0
[d]: https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9
[e]: https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22

[f]: https://github.com/r-pufky/ansible_users/blob/main/LICENSE
[g]: https://github.com/r-pufky/ansible_collection_deb
[i]: https://github.com/r-pufky/ansible_users/blob/main/meta/main.yml
[j]: https://github.com/r-pufky/ansible_users/tree/main/defaults/main/main.yml
[k]: https://github.com/r-pufky/ansible_users/tree/main/defaults/main/account_definitions.yml
[l]: https://arlimus.github.io/articles/usepam
[m]: https://unix.stackexchange.com/questions/193066/