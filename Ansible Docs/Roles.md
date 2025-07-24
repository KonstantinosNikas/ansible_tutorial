In **Ansible**, a **role** is a structured way to organize playbooks and tasks. It allows you to break down complex configurations into reusable and modular components. Roles promote reuse, clarity, and maintainability.

A **role** is essentially a directory structure that defines a set of related tasks, variables, files, templates, and handlers, bundled together.

Roles are used by referencing them in a playbook using the `roles:` directive.

**Δημιουργία φακέλου `roles`**
```
mkdir roles
cd roles
```
Αυτός ο φάκελος θα περιέχει όλους τους ρόλους που θέλουμε να εφαρμόσουμε σε διαφορετικά hosts.

**Δημιουργία φακέλων για κάθε ρόλο (role)**
```
mkdir base
mkdir db_servers
mkdir file_servers
mkdir web_servers
mkdir workstations
```
Κάθε φάκελος αντιπροσωπεύει έναν ρόλο που εφαρμόζεται σε συγκεκριμένους τύπους servers.

**Δημιουργία του υποφακέλου `tasks` σε κάθε ρόλο**

```
cd base
mkdir tasks
cd ../db_servers && mkdir tasks
cd ../file_servers && mkdir tasks
cd ../web_servers && mkdir tasks
cd ../workstations && mkdir tasks
```

Ο φάκελος `tasks/` περιέχει το βασικό αρχείο `main.yml` που περιγράφει τι πρέπει να εκτελέσει ο ρόλος.

**ΔΗΜΙΟΥΡΓΙΑ ΤΩΝ `main.yml` ΑΡΧΕΙΩΝ ΑΝΑ ΡΟΛΟ**
`base/tasks/main.yml`
```
- name: add ssh key for simone
  authorized_key:
    user: simone
    key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAe7/ofWLNBq3+fRn3UmgAizdicLs9vcS4Oj8VSOD1S/ ansible"
```
Προσθέτει το SSH public key στον χρήστη `simone`.

`db_servers/tasks/main.yml`
```
- name: install mariadb server package (CentOS)
  tags: centos,db,mariadb
  dnf:
    name: mariadb
    state: latest
  when: ansible_distribution == "CentOS"

- name: install mariadb server
  tags: db,mariadb,ubuntu
  apt:
    name: mariadb-server
    state: latest
  when: ansible_distribution == "Ubuntu"

```
Εγκαθιστά MariaDB σε CentOS ή Ubuntu, ανάλογα με το λειτουργικό.

`file_servers/tasks/main.yml`
```
- name: install samba package
  tags: samba
  package:
    name: samba
    state: latest
```
Εγκαθιστά το Samba (file sharing).

`workstations/tasks/main.yml`
```
- name: install unzip
  package:
    name: unzip

- name: install terraform
  unarchive:
    src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
    dest: /usr/local/bin
    remote_src: yes
    mode: 0755
    owner: root
    group: root

```
Εγκαθιστά `unzip` και αποσυμπιέζει το `terraform` στο `/usr/local/bin`.

`web_servers/tasks/main.yml`
```
- name: install httpd package (CentOS)
  tags: apache,centos,httpd
  dnf:
    name:
      - httpd
      - php
    state: latest
  when: ansible_distribution == "CentOS"

- name: start and enable httpd (CentOS)
  tags: apache,centos,httpd
  service:
    name: httpd
    state: started
    enabled: yes
  when: ansible_distribution == "CentOS"

- name: install apache2 package (Ubuntu)
  tags: apache,apache2,ubuntu
  apt:
    name:
      - apache2
      - libapache2-mod-php
    state: latest
  when: ansible_distribution == "Ubuntu"

- name: change e-mail address for admin
  tags: apache,centos,httpd
  lineinfile:
    path: /etc/httpd/conf/httpd.conf
    regexp: '^ServerAdmin'
    line: ServerAdmin somebody@somewhere.net
  when: ansible_distribution == "CentOS"
  register: httpd

- name: restart httpd (CentOS)
  tags: apache,centos,httpd
  service:
    name: httpd
    state: restarted
  when: httpd.changed

- name: copy html file for site
  tags: apache,apache2,httpd
  copy:
    src: default_site.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: 0644

```
Εγκαθιστά Apache (ανά OS), ρυθμίζει το email admin, επανεκκινεί την υπηρεσία αν άλλαξε κάτι και ανεβάζει HTML αρχείο.

ΤΟ ΚΕΝΤΡΙΚΟ PLAYBOOK: `playbook_example.yml`
```
---
- hosts: all
  become: true
  pre_tasks:
    - name: update repository index (CentOS)
      tags: always
      dnf:
        update_cache: yes
      changed_when: false
      when: ansible_distribution == "CentOS"

    - name: update repository index (Ubuntu)
      tags: always
      apt:
        update_cache: yes
      changed_when: false
      when: ansible_distribution == "Ubuntu"

- hosts: all
  become: true
  roles:
    - base

- hosts: workstations
  become: true
  roles:
    - workstations

- hosts: web_servers
  become: true
  roles:
    - web_servers

- hosts: db_servers
  become: true
  roles:
    - db_servers

- hosts: file_servers
  become: true
  roles:
    - file_servers

```
Εκτελεί:

1. Προ-εργασίες (update repo cache)
    
2. Ρόλο `base` για όλους
    
3. Ειδικούς ρόλους ανά ομάδα hosts (π.χ. μόνο τα `web_servers` τρέχουν τον web role)

ΤΡΕΞΙΜΟ PLAYBOOK
```
ansible-playbook site.yml
```
