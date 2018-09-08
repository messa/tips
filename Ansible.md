Ansible tips
============

https://www.ansible.com/

https://www.root.cz/serialy/konfiguracni-a-orchestracni-nastroj-ansible/ (but it's from 2013)

https://leucos.github.io/ansible-files-layout

Terminology
-----------

- Role
  - focuses on very specific goal, like "install nginx"
  
- Playbook
  - you basically tell Ansible: _please install roles foo, bar and baz on machines alice, bob and charlie._

- Inventory
  - list of hosts and host groups (the group `all` automatically contains all hosts)
  - inventories can also come with variables applied to hosts or groups (including all)
  - inventories can be dynamic. If the inventory file is executable, Ansible will run it and use its output as the inventory
  - you can have multiple inventories
    -  for example when you manage servers for different customers


File structure
--------------

Role layout:

- all directories are optional besides `tasks`
- for each directory, the entry point is `main.yml`

```
ansible-foobar/
├── defaults         - defaults for variables used in roles
│   └── main.yml
├── files            - files that do not require Jinja processing
├── handlers         - where you define handlers that get notified by tasks
│   └── main.yml
├── meta
│   └── main.yml     - only two variables: galaxy_info, dependencies
├── tasks
│   ├── foobar.yml
│   └── main.yml
└── templates        - files processed by Jinja (variables are interpreted etc.)
    └── foobar.conf.j2
```

Inventories and playbook layout:

```
playbook-foobar/
├── ansible.cfg
├── requirements.yml
├── .imported_roles/
├── inventories
│   ├── development
│   |   ├── group_vars
│   |   │   └── all
│   |   └── hosts
│   ├── integration
│   |   ├── group_vars
│   |   │   └── all
│   |   └── hosts
│   └── production
│       ├── group_vars
│       │   └── all
│       └── hosts    
├── site.yml
└── playbooks
    ├── database.yml
    └── stuff.yml
```    

`ansible.cfg`

```
hostfile = ./inventories/dev
roles_path = ./.imported_roles:/some/dev/place/with/roles
```

Task-only playbooks vs. roles
    
