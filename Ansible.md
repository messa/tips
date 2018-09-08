Ansible tips
============

https://www.ansible.com/

https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html

https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html

https://pschiffe.github.io/ansible-docker/

https://www.root.cz/serialy/konfiguracni-a-orchestracni-nastroj-ansible/ (but it's from 2013)

https://leucos.github.io/ansible-files-layout

https://github.com/pschiffe/ansible-docker/blob/master/examples/rocketchat/rocketchat.yml

https://docs.ansible.com/ansible/latest/modules/docker_container_module.html

https://docs.ansible.com/ansible/latest/user_guide/vault.html

https://docs.ansible.com/ansible/latest/user_guide/playbooks_vault.html

http://www.mydailytutorials.com/ansible-create-files/

https://docs.ansible.com/ansible/2.5/modules/file_module.html

https://www.digitalocean.com/community/tutorials/how-to-use-vault-to-protect-sensitive-ansible-data-on-ubuntu-16-04

https://gist.github.com/tristanfisher/e5a306144a637dc739e7

https://stackoverflow.com/questions/26638180/write-variable-to-a-file-in-ansible

https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings-locations

https://github.com/messa/pyladies-courseware/blob/deployment/deployment/playbooks/setup_nginx.yml


Terminology
-----------

- Task

- Role
  - focuses on very specific goal, like "install nginx"
  
- Play

  ```yaml
  - hosts: webservers
  roles:
     - common
     - webservers
  ```
  
- Playbook
  - you basically tell Ansible: _please install roles foo, bar and baz on machines alice, bob and charlie._

- Strategy

- Inventory
  - list of hosts and host groups (the group `all` automatically contains all hosts)
  - inventories can also come with variables applied to hosts or groups (including all)
  - inventories can be dynamic. If the inventory file is executable, Ansible will run it and use its output as the inventory
  - you can have multiple inventories
    -  for example when you manage servers for different customers


File structure
--------------

Example project structure (from [docs](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)):

```
site.yml
webservers.yml
fooservers.yml
roles/
   common/      - a role
     tasks/     - main list of tasks to be executed by the role
     handlers/  - handlers, may be used by this role or even anywhere outside this role
     files/     - files which can be deployed via this role
     templates/ - templates which can be deployed via this role
     vars/      - other variables for the role
     defaults/  - default variables for the role
     meta/      - some meta data for this role (dependencies etc.)
   webservers/  - another role
     tasks/
     defaults/
     meta/
```

Each directory must contain a `main.yml` file.

Other YAML files may be included in certain directories – for example, it is common practice to have platform-specific tasks included from the `tasks/main.yml` file:

```yaml
# roles/example/tasks/main.yml
- name: added in 2.4, previously you used 'include'
  import_tasks: redhat.yml
  when: ansible_os_platform|lower == 'redhat'
- import_tasks: debian.yml
  when: ansible_os_platform|lower == 'debian'

# roles/example/tasks/redhat.yml
- yum:
    name: "httpd"
    state: present

# roles/example/tasks/debian.yml
- apt:
    name: "apache2"
    state: present
```




---


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
    
