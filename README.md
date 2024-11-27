Ansible Role: sudo
==================

[![Galaxy Role][badge-galaxy]][link-galaxy]
[![MIT licensed][badge-license]][link-license]
[![CI][badge-gh-actions]][link-gh-actions]

Configure sudo policy for Linux and FreeBSD.

At minimum, you should specify one or more users in the `sudo_users` list. This will add the users to the default sudo group for the system. This group will allow users to run any command on the system as root. If you would like to restrict access to specific commands, use the `sudo_rules` list to specify any sudo access rules. Aliases can also be defined with the `sudo_<type>_aliases` lists.

Global sudo defaults are defined in `sudo_defaults`. This list can be overridden to change defaults. For more granular defaults, the list `sudo_per_object_defaults` can be used to change defaults for particular users or commands.

Installation
------------

Using Ansible Galaxy: `ansible-galaxy install wesmarcum.sudo`

Requirements
------------

No other Ansible roles are required.

Role Variables
--------------

| Variable Name            | Default Value         | Description                                                                                           |
|--------------------------|-----------------------|-------------------------------------------------------------------------------------------------------|
| sudo_group               | "sudo"                | Group to create and grant full sudo access. Members of this group have full sudo permissions.         |
| sudo_group_nopasswd      | false                 | If 'true', don't require a password for the group defined by 'sudo_group'.                            |
| sudo_users               | empty                 | List of users to add to the 'sudo_group'.                                                             |
| sudo_editor_path         | empty                 | Path to your preferred editor for `visudo`.                                                           |
| sudo_defaults            | see defaults/main.yml | List of default global sudo options.                                                                  |
| sudo_per_object_defaults | see defaults/main.yml | List of defaults to apply per 'object'. An object can be a user, group, host, command, or runas user. |
| sudo_keep_proxy          | true                  | Keep proxy environment variables for the user running sudo.                                           |
| sudo_secure_path         | see defaults/main.yml | Path to preserve in the environment for trusted executables.                                          |
| sudo_log_output          | false                 | Log output for `sudoreplay`.                                                                          |
| sudo_user_aliases        | empty                 | List of user aliases for sudo rules.                                                                  |
| sudo_command_aliases     | empty                 | List of command aliases for sudo rules.                                                               |
| sudo_host_aliases        | empty                 | List of host aliases for sudo rules.                                                                  |
| sudo_runas_aliases       | empty                 | List of runas aliases for sudo rules.                                                                 |
| sudo_rules               | empty                 | List of custom sudo access rules.                                                                     |

Example Playbook
----------------

Simple playbook to setup the sudo group and add a user:

```yaml
- hosts: all
  roles:
    - role: wesmarcum.sudo
      vars:
        sudo_group: "sudo"
        sudo_users:
          - myuser
```

Playbook with all options commented. The `sudo` role can be used to set up defaults, aliases, and sudo rule sets.

```yaml
- hosts: all
  roles:
    - role: wesmarcum.sudo
      vars:
        sudo_users:
          - myuser
        # Log output for sudoreplay.
        sudo_log_output: true
        # Set up defaults per user, group, host, command, or 'runas' user.
        sudo_per_object_defaults:
          - type: user
            name: test
            options:
              - "!lecture"
              - "!insults"
              - timestamp_timeout=10
          - type: group
            name: wheel
            options:
              - timestamp_timeout=2
          - type: host
            name: PRODUCTION
            options:
              - lecture=always
          - type: command
            name: "/sbin/fdisk"
            options:
              - lecture=always
          - type: runas
            name: "test"
            options:
              - lecture
        # User aliases for sudo rules.
        sudo_user_aliases:
          - name: BACKUPOPERATORS
            users:
              - backup
              - devops
          - name: TEST
            users:
              - molecule
              - bob
        # Command aliases.
        sudo_command_aliases:
          - name: BACKUP
            commands:
              - /sbin/dump
              - /sbin/restore
              - /usr/bin/mount
        # Host aliases.
        sudo_host_aliases:
          - name: WWW
            hosts:
              - www[0-3]
          - name: DMZ
            hosts:
              - 192.168.50.0/24
              - 192.168.51.0/255.255.255.0
              - WWW
        # Runas aliases.
        sudo_runas_aliases:
          - name: DB
            users:
              - postgres
              - mysql
        # Custom sudo rules.
        sudo_rules:
          - name: "%sudo"
            host: ALL
            runas_user: ALL
            runas_group: ALL
            tag: NOPASSWD
            commands:
              - ALL
          - name: devops
            host: DMZ
            runas_user: root
            commands:
              - BACKUP
          - name: devops
            host: DMZ
            runas_user: root
            tag: NOPASSWD
            commands:
              - /sbin/reboot
              - /sbin/shutdown
```

Compatibility
-------------

| OS         | Version      |
|------------|--------------|
| Arch Linux | all          |
| Debian     | 10, 11, 12   |
| FreeBSD    | 13.1         |
| Ubuntu     | 20.04, 22.04, 24.04 |

License
-------

MIT

Author Information
------------------

<https://github.com/wesmarcum>

[badge-license]: https://img.shields.io/badge/license-MIT-green?
[link-license]: https://github.com/wesmarcum/ansible-role-sudo/blob/main/LICENSE
[badge-gh-actions]: https://github.com/wesmarcum/ansible-role-sudo/workflows/CI/badge.svg?event=push
[link-gh-actions]: https://github.com/wesmarcum/ansible-role-sudo/actions?query=workflow%3ACI
[badge-galaxy]: https://img.shields.io/badge/role-sudo-blue
[link-galaxy]: https://galaxy.ansible.com/ui/standalone/roles/wesmarcum/sudo/
