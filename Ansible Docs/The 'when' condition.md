```
---

- name: Install Apache with PHP support on all hosts
  hosts: all
  become: true

  tasks:
    - name: Update APT package index
      apt:
        update_cache: yes
      when: ansible_distribution == "Debian"

    - name: Install Apache2
      apt:
        name: apache2
        state: latest

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
### `when:`

Είναι το **conditional keyword** της Ansible — επιτρέπει να εκτελείται ένα task **μόνο αν** ισχύει μια συνθήκη.
### `ansible_distribution`

Είναι **ένα built-in Ansible fact** (ένα στοιχείο που συλλέγει αυτόματα), το οποίο επιστρέφει το όνομα της διανομής Linux, π.χ.:

|OS|ansible_distribution|
|---|---|
|Debian|`"Debian"`|
|Ubuntu|`"Ubuntu"`|
|CentOS|`"CentOS"`|
|Rocky Linux|`"Rocky"`|

Μπορείς να δεις όλα τα facts τρέχοντας:
`ansible all -m setup`

Αν  θέλω να περιλάβω περισσότερα  distributions που χρησιμοποιουν την ίδια εντολή τότε η φράση αλλάζει σε 

`when: ansible_distribution in ["Debian", "Ubuntu"]`

Αν θέλω να τρέξω τις αλλαγές σε διαφορετικά  distributions, δηλαδή πχ που χρησιμοποιούν διαφορετικές εντολές εγκατάστασης.

```
---

- name: Install Apache with PHP support on all hosts
  hosts: all
  become: true

  tasks:
    - name: Update APT package index
      apt:
        update_cache: yes
      when: ansible_distribution == "Debian"

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

- name: Update APT package index
      dnf:
        update_cache: yes
      when: ansible_distribution == "CentoOS"

    - name: Install Apache2
      dnf:
        name: apache2
        state: latest
      when: ansible_distribution == "CentoOS"

    - name: Add PHP support for Apache
      dnf:
        name:
          - httpd
          - php
        state: latest
      when: ansible_distribution == "CentoOS"
```
