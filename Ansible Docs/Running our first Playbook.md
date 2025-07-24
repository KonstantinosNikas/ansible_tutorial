Για εγκατάσταση των παρακάτω φτιάχνουμε το playbook.
install_apache.yml
```
---

- name: Install Apache on all hosts
  hosts: all
  become: true

  tasks:
   - name: update repository index
     apt:
       update_cache: true

   - name: install apache2 package
     apt:
       name: apache2
       state: latest
       update_cache: true

   - name: add php support for apache
     apt:
       name: apache2
       state: latest
       update_cache: true

```
- Το playbook θα εκτελεστεί σε **όλους τους hosts** (`hosts: all`).
    
- Θα χρησιμοποιήσει **`sudo` δικαιώματα** (`become: true`), δηλαδή θα εκτελεστεί ως root.

Για την αφαίρεση τους.
remove_apache.yml

```
---

- name: Install Apache with PHP support on all hosts
  hosts: all
  become: true
  tasks:
    - name: Update APT package index
      apt:
        update_cache: yes


    - name: Install Apache2
      apt:
        name: apache2
        state: absent

    - name: Add PHP support for Apache
      apt:
        name:
          - libapache2-mod-php
          - php
        state: absent
```
