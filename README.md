# Ansible playbooks

Creating local users and groups on multiple remote servers.

## Directory structure
```
├── groups.yml
├── README.md
├── remove_groups.yml
├── remove_users.yml
├── site.yml
└── roles
    ├── inventory.yml
    ├── meta
    │   └── test_passwords
    ├── tasks
    │   └── sudoers.yml
    └── vars.yml
```
4 directories, 9 files

The main playbook is site.yml. Run the command below:
```
ansible-playbook site-yml -i inventory
```

site.yml playbook
```
- import_playbook: groups.yml

- hosts: all
  become: true
  tasks:

  - name: Importing users' variables from file
    include_vars:
      file: roles/vars.yml

  - name: Adding local sysadmin users to multiple distro servers
    ansible.builtin.user:
      name: "{{ item.name }}"
      password: "{{ item.password }}"
      comment: "{{ item.comment }}"
      group: sysadmin
      groups: storageadmin,localuser
      append: yes
      shell: /bin/bash
      state: present
      generate_ssh_key: yes
      ssh_key_bits: 2048
      ssh_key_file: .ssh/id_rsa
    loop: "{{ users_list }}"

  - name: Changing users' home directory permission mode
    shell: chmod 750 /home/{{ item.name }}
    loop: "{{ users_list }}"

  - name: Editing all sudoers files to NOPASSWD for sysadmin group
    include_tasks: roles/tasks/sudoers.yml
```
