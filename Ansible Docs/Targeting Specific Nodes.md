Μπορούμε να χωρίσουμε τις IP σε διαφορετικούς κόμβους μέσα στο inventory.

```
[myservers]
83.212.72.64

[dbservers]
83.212.72.65

[fileservers]
83.212.72.66
```

 Ένα Playbook θα άλλαζε και θα γινόταν κάπως έτσι:
```
---

- name: Update all systems
  hosts: all
  become: true

  tasks:
    - name: Install system updates
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
```

#### `pre_tasks`
In Ansible, `pre_tasks` are **special tasks that run before the main `tasks` section** of a play. They're useful when you want to perform setup steps **before** the actual configuration starts.
```
---

- name: Update all systems
  hosts: all
  become: true

  pre_tasks:
    - name: Install system updates
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
  
- hosts: db_servers
  become: true
  tasks:

    - name: install mariaDB package (debian)
      apt: 
        name: mariadb-server
        state: latest
      when: ansible_distribution == "Debian"

- hosts: file_servers
  become: true
  tasks:

    - name: install samba package
      package: 
        name: samba
        state: latest
```
