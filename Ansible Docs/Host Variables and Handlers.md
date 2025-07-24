Ο σκοπός των **Host Variables** και των **Handlers** στο Ansible είναι να κάνουν τις playbooks και τους ρόλους σου **πιο ευέλικτους**, **επαναχρησιμοποιήσιμους** και **εύκολα διαχειρίσιμους**, ειδικά όταν δουλεύεις με πολλούς διαφορετικούς servers ή διανομές.

Οι **Handlers** είναι ειδικά tasks που εκτελούνται **μόνο όταν ειδοποιούνται** από άλλα tasks με `notify`.

Για τις host_variables δημιουργούμε ένα directory και όσα files θέλουμε για να βάλουμε μεταβλητές
```
cd host_vars

nano 83.212.72.64.yml 

# Περιεχόμενο 83.212.72.64.yml 

apache_package_name: apache2
apache_service: apache2
php_package_name: libapache2-mod-php

# Αν είχα για παράδειγμα κι ένα μηχάκι με CentOS
# Περιεχόμενο centos-web-01.yml

apache_package_name: httpd
apache_service: httpd
php_package_name: php


```

Οι μεταβλητές αυτές καθορίζουν **ποιο πακέτο ή υπηρεσία θα χρησιμοποιήσει ο ρόλος**, ανάλογα με το OS.

#### MAIN.YML (WEB_SERVERS ROLE)
```
- name: install web server packages
  tags: apache,apache2,centos,httpd,ubuntu
  package:
    name:
      - "{{ apache_package_name }}"
      - "{{ php_package_name }}"
    state: latest

- name: start and enable apache
  tags: apache,centos,httpd
  service:
    name: "{{ apache_service }}"
    state: started
    enabled: yes

- name: change e-mail address for admin
  tags: apache,centos,httpd
  lineinfile:
    path: /etc/httpd/conf/httpd.conf
    regexp: '^ServerAdmin'
    line: ServerAdmin somebody@somewhere.net
  when: ansible_distribution == "CentOS"
  notify: restart_apache

- name: copy html file for site
  tags: apache,apache2,httpd
  copy:
    src: default_site.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: 0644

```

Για να δημιουργήσουμε ένα **handler** πάμε μέσα σε ένα ρόλο πχ /roles/web_servers/

```
mkdir handlers

nano main.yml

- name: restart_apache
  tags: apache,centos,httpd
  service:
    name: "{{ apache_service }}"
    state: restarted


```
##### Τι κάνει;

- Επανεκκινεί την υπηρεσία του Apache web server (π.χ. `httpd` ή `apache2`).
    
- Χρησιμοποιεί τη μεταβλητή `apache_service` για να καθορίσει το σωστό όνομα υπηρεσίας, ανάλογα με το λειτουργικό σύστημα.
##### Πότε εκτελείται αυτός ο handler;
Εκτελείται **μόνο αν ειδοποιηθεί από κάποιο task** με:
`notify: restart_apache`
