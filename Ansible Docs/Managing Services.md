### Τι είναι η διαχείριση υπηρεσιών στο Ansible;

Με το Ansible μπορείς να **ξεκινήσεις**, **σταματήσεις**, **επανακινήσεις**, ή να **ενεργοποιήσεις/απενεργοποιήσεις**υπηρεσίες (services) σε απομακρυσμένους υπολογιστές, μέσω του module `service` ή `systemd`.

Παράδειγμα βασικού `task` για υπηρεσία:
```
- name: Ξεκίνα και ενεργοποίησε την υπηρεσία apache2
  ansible.builtin.service:
    name: apache2         # Όνομα της υπηρεσίας
    state: started        # Ξεκινά την υπηρεσία
    enabled: yes          # Να ξεκινά αυτόματα στο boot

```
### Τιμές για `state` (κατάσταση υπηρεσίας):

| Τιμή `state` | Περιγραφή                                  |
| ------------ | ------------------------------------------ |
| `started`    | Ξεκινά την υπηρεσία αν δεν τρέχει ήδη      |
| `stopped`    | Σταματά την υπηρεσία                       |
| `restarted`  | Κάνει επανεκκίνηση                         |
| `reloaded`   | Κάνει reload τη ρύθμιση χωρίς επανεκκίνηση |
**Αλλαγή email διαχειριστή**
```
- name: change e-mail address for admin
  tags: apache,centos,httpd
  lineinfile:
    path: /etc/httpd/conf/httpd.conf   
    regexp: '^ServerAdmin'
    line: ServerAdmin somebody@somewhere.net
  when: ansible_distribution == "CentOS"

```
**Τι κάνει (όταν διορθωθεί):**  
– Βρίσκει τη γραμμή που ξεκινά με `ServerAdmin` στο αρχείο `httpd.conf`  
– Την αλλάζει ώστε να περιέχει το νέο email: `somebody@somewhere.net`

Στο **Ansible**, η λέξη-κλειδί `register` χρησιμοποιείται για να **αποθηκεύσεις την έξοδο** ενός task (εντολής) σε μια μεταβλητή, ώστε να μπορείς να τη χρησιμοποιήσεις αργότερα στο playbook σου.

```
- name: Έλεγξε αν τρέχει η υπηρεσία nginx
  ansible.builtin.shell: systemctl is-active nginx
  register: nginx_status

```

Παράδειγμα 
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

- name: Setup workstations
  hosts: workstations
  become: true

  tasks: 
    - name: Install unzip
      package:
        name: unzip
        state: present

    - name: Install terraform
      unarchive:
        src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
        dest: /usr/local/bin
        remote_src: yes
        mode: '0755'
        owner: root
        group: root

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

    - name: Check if Apache is active
      command: systemctl is-active apache2
      register: apache_status

    - name: Show Apache status
      debug:
        msg: "Apache status: {{ apache_status.stdout }}"

    - name: Ensure Apache is running and enabled
      service:
        name: apache2
        state: started
        enabled: yes
      when: apache_status.stdout != "active"

    - name: Set ServerAdmin email in config
      lineinfile:
        path: /etc/apache2/apache2.conf
        regexp: '^ServerAdmin'
        line: 'ServerAdmin admin@example.com'
        backup: yes
      when: ansible_distribution == "Debian"

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

    - name: Check if MariaDB is active
      command: systemctl is-active mariadb
      register: mariadb_status
      when: ansible_distribution == "Debian"

    - name: Show MariaDB status
      debug:
        msg: "MariaDB status: {{ mariadb_status.stdout }}"
      when: mariadb_status is defined

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

    - name: Add custom comment in smb.conf
      lineinfile:
        path: /etc/samba/smb.conf
        line: '# Installed via Ansible playbook'
        insertafter: '[global]'
        state: present

```

## Χρήση `register`

Το `register` χρησιμοποιείται για να **αποθηκεύσουμε την έξοδο** από ένα task, ώστε να μπορούμε να την ελέγξουμε ή να την εμφανίσουμε ή να δράσουμε ανάλογα σε επόμενο βήμα.

Τι κάνει:
- Εκτελεί την εντολή `systemctl is-active apache2` στο απομακρυσμένο σύστημα.
    
- Αποθηκεύει το αποτέλεσμα (π.χ. "active", "inactive", "failed") στη μεταβλητή `apache_status`.

## Χρήση `lineinfile`

Το module `lineinfile` χρησιμοποιείται για να **επεξεργαστούμε ή να προσθέσουμε** μια συγκεκριμένη γραμμή μέσα σε ένα αρχείο κειμένου στο απομακρυσμένο σύστημα.

Τι κάνει:
- Εντοπίζει τη γραμμή που ξεκινά με `ServerAdmin` στο αρχείο `apache2.conf`.
    
- Αν τη βρει, την **αντικαθιστά** με `ServerAdmin admin@example.com`.
    
- Αν **δεν υπάρχει**, την προσθέτει.
    
- Με `backup: yes` κρατάει αντίγραφο ασφαλείας του αρχείου πριν την αλλαγή.