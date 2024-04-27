# Users
Manage users and groups in a sane and programmatic way for easy role and user
consumption.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_users/blob/main/meta/main.yml)

[collections/roles](https://github.com/r-pufky/ansible_users/blob/main/meta/requirements.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_users/tree/main/defaults/main/)

## Dependencies
N/A

## Example Playbook
Main role is currently a NO-OP until the rest of the private repository is
migrated to ansible 2.16 and moved into the `r_pufky.srv` collection.

### Role Playbook
Roles many consume this role to easily create mandatory users and groups
required for execution. Run once for each user/group to create.

roles/my_custom_role/tasks/add_mandatory_users.yml
``` yaml
- name: 'add required service user/groups'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.users'
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

## Unit Testing
Test framework requires molecule and rootless podman setup.

Run all unit tests:
``` bash
molecule test --all
```

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_users/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
