### What Are **Ansible Tags**?

**Ansible tags** let you **label tasks, roles, or blocks** in your playbook so you can **run only specific parts** of the playbook when you want — without executing the whole thing.

```
---
- name: Update all systems
  hosts: all
  become: true

  pre_tasks:
    - name: Install system updates
      tags: always
      apt:
        upgrade: dist
        update_cache: yes
      when: ansible_distribution == "Debian"

# --------------------------------------------

- name: Install Apache with PHP on webservers
  hosts: webservers
  become: true

  tasks:
    - name: Install Apache2
      tags: [apache, apache2, debian]
      apt:
        name: apache2
        state: latest
      when: ansible_distribution == "Debian"

    - name: Add PHP support for Apache
      apt:
        name:
          - libapache2-mod-php
          - php
        state: latest
      when: ansible_distribution == "Debian"

    - name: Ensure Apache is running and enabled
      service:
        name: apache2
        state: started
        enabled: yes

# --------------------------------------------

- name: Install MariaDB on db servers
  hosts: db_servers
  become: true

  tasks:
    - name: Install MariaDB package (Debian)
      tags: [db, maria, debian]
      apt:
        name: mariadb-server
        state: latest
      when: ansible_distribution == "Debian"

# --------------------------------------------

- name: Install Samba on file servers
  hosts: file_servers
  become: true

  tasks:
    - name: Install samba package
      tags: [samba]
      package:
        name: samba
        state: latest

```
Για να δώ τα tags τρέχω:
`ansible-playbook --list-tags site.yml`

 Για να τρέξω ένα tag:
 `ansible-playbook site.yml --tags apache`


 
