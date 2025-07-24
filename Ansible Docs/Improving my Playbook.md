Μπορούμε να βελτιώσουμε το playbook μας συγχωνεύοντας γραμμές και κάνοντας το πιο ευκολο στο διάβασμα.
```
---
- name: Install Apache with PHP support (Debian-based)
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Install and configure Apache with PHP
      apt:
        name:
          - apache2
          - libapache2-mod-php
          - php
        state: latest
        update_cache: yes
      when: ansible_distribution in ["Debian", "Ubuntu"]

    - name: Ensure Apache is running and enabled
      service:
        name: apache2
        state: started
        enabled: yes

```
Άλλη μια εκδοχή πιο δύσκολη:

```
---
- name: Install Apache with PHP support on Debian-based systems
  hosts: all
  become: true
  gather_facts: true
  tasks:
    - name: Install Apache and PHP (Debian/Ubuntu)
      apt:
        name:
          - apache2
          - libapache2-mod-php
          - php
        state: latest
        update_cache: yes
      when: ansible_distribution in ["Debian", "Ubuntu"]

    - name: Ensure Apache is running and enabled (Debian/Ubuntu)
      service:
        name: apache2
        state: started
        enabled: yes
      when: ansible_distribution in ["Debian", "Ubuntu"]

- name: Install Apache with PHP support on RHEL-based systems
  hosts: all
  become: true
  gather_facts: true
  tasks:
    - name: Install Apache and PHP (CentOS/RedHat/Rocky)
      dnf:
        name:
          - httpd
          - php
        state: latest
        update_cache: yes
      when: ansible_distribution in ["CentOS", "RedHat", "Rocky", "AlmaLinux"]

    - name: Ensure Apache is running and enabled (CentOS/RedHat/Rocky)
      service:
        name: httpd
        state: started
        enabled: yes
      when: ansible_distribution in ["CentOS", "RedHat", "Rocky", "AlmaLinux"]

```
Η καλύτερη μέθοδος να δηλώσουμε στο inventory τις μεταβλητές πχ
```
[debian_servers]
debian1 ansible_host=192.168.1.10 apache_pkg=apache2 php_pkgs="['libapache2-mod-php', 'php']" svc_name=apache2

[redhat_servers]
centos1 ansible_host=192.168.1.11 apache_pkg=httpd php_pkgs="['php']" svc_name=httpd

[all:vars]
ansible_user=debian
ansible_ssh_private_key_file=~/.ssh/ansible_key

```
και το playbook θα είναι κάπως έτσι:
```
- name: Install Apache with PHP support
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Install Apache and PHP
      package:
        name: "{{ [apache_pkg] + php_pkgs }}"
        state: latest
        update_cache: yes

    - name: Ensure Apache is running and enabled
      service:
        name: "{{ svc_name }}"
        state: started
        enabled: yes


```
