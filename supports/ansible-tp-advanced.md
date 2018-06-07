<!-- $theme: gaia -->
<!-- $size: a4 -->
<!-- template: invert -->

# Ansible Training Advanced
![center](assets/ansible_logo.png)

---
# Agenda

- Deploy 3 VMs on Scaleway

- Set up a three tiers Wordpress installation

	- Nginx reverse proxy
	- Nginx + PHP FPM Wordpress
	- Mysql backend

---
### Create your Ubuntu VM (1)

```
user@laptop:~#$ tree
├── playbook.yml
└── roles
    └── scaleway_vm
        └── tasks
            └── main.yml
```
Ubuntu image: `e20532c4-1fa0-4c97-992f-436b8d372c07`
Organization: `43a3b6c8-916f-477b-b7ec-ff1898f5fdd9`

Commercial Type: `VC1S` - Location: `par1`

⚠️ Specify a custom name (ie not ansible/test/...) ⚠️

[Module Documentation](http://docs.ansible.com/ansible/devel/modules/scaleway_compute_module.html#scaleway-compute-module)

---

### Create your Ubuntu VM (2)

```
user@laptop:~#$ tree
├── playbook.yml
└── roles
    └── scaleway_vm
        └── tasks
            └── main.yml
```
Ubuntu image: `6d7aabd0-a0b7-434a-95c8-b40aa3d5b973`
Organization: `43a3b6c8-916f-477b-b7ec-ff1898f5fdd9`

Commercial Type: `VC1S` - Location: `ams1`

⚠️ Specify a custom name (ie not ansible/test/...) ⚠️

[Module Documentation](http://docs.ansible.com/ansible/devel/modules/scaleway_compute_module.html#scaleway-compute-module)

---

### Solution

```
#roles/scaleway/tasks/main.yml
- name: deploy ssh key to scaleway
  scaleway_sshkey:
    ssh_pub_key: "ssh-rsa ..."
    state: present

- name: Create front, middle, back scaleway servers
  scaleway_compute:
    name: "{{ item }}"
    state: "{{ state }}"
    image: "{{ image }}"
    organization: "{{ organization }}"
    region: "{{ location }}"
    commercial_type: "{{ commercial_type }}"
  with_items:
    - front-nginx
    - middle-wordpress
    - back-mysql

```

---
# Create your inventory file

Get the IP Address related to your instance and create your inventory file.
*You can take example on `/etc/ansible/hosts`*

Test your inventory file:

```
user@laptop:~#:~# ansible -i inventory all -m ping
51.15.235.20 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
**Make sure to connect with root user with `ansible_user`**

---

### Solution

```
[front]
<SCW_IP> ansible_user=root

[middle]
<SCW_IP> ansible_user=root

[back]
<SCW_IP> ansible_user=root

```

---

# Nginx role

---
### Create an nginx role and create a new playbook

```
[ansible01@ansible ~]$ tree
├── inventory
├── playbook_wordpress.yml
└── roles
    ├── nginx_install
        ├── files
        │   └── nginx.conf
        ├── handlers
        │   └── main.yml
        ├── tasks
        │   └── main.yml
        └── templates
            └── nginx-wordpress.conf
```

Try: `ansible-playbook -i inventory playbook.yml`

---
### Solution

[Nginx Role](https://github.com/antoineHC/ansible-meetup-solutions/tree/master/solution_wordpress/roles/nginx_install)

[Playbook](https://github.com/antoineHC/ansible-meetup-solutions/blob/master/solution_wordpress/playbook_wordpress.yml)

---

# Wordpress role

---
### Create a Wordpress role

```
[ansible01@ansible ~]$ tree
├── group_vars
│   └── all
├── inventory
├── playbook_wordpress.yml
└── roles
    └── wordpress_install
        ├── files
        │   ├── nginx.conf
        │   └── wordpress.conf
        ├── handlers
        │   └── main.yml
        ├── tasks
        │   └── main.yml
        └── templates
            └── wp-config.php
```
---

---
### Solution

[Wordpress Role](https://github.com/antoineHC/ansible-meetup-solutions/tree/master/solution_wordpress/roles/wordpress_install)

---

# Mysql role

---
### Create a Mysql role

```
[ansible01@ansible ~]$ tree
├── group_vars
│   └── all
├── inventory
├── playbook_scw.yml
├── playbook_wordpress.yml
└── roles
    ├── mysql_install
        ├── defaults
        │   └── main.yml
        ├── handlers
        │   └── main.yml
        ├── tasks
        │   └── main.yml
        └── templates
            └── my.cnf
```
---

---
### Solution

[Mysql Role](https://github.com/antoineHC/ansible-meetup-solutions/tree/master/solution_wordpress/roles/mysql_install)

---

# Thanks