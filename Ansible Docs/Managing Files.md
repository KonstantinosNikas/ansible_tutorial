Managing files with Ansible is one of its core strengths — you can **copy**, **create**, **modify**, **delete**, or **template** files on remote hosts using built-in modules.

## Common Modules for File Management

| Module        | What it does                                        | Example use               |
| ------------- | --------------------------------------------------- | ------------------------- |
| `copy`        | Copies a file from control node → remote            | Push a config file        |
| `template`    | Renders a Jinja2 template with variables            | Dynamic config like Nginx |
| `file`        | Manages file/directory permissions, ownership, type | Create dirs, symlinks     |
| `lineinfile`  | Ensures a line exists in a file                     | Add config lines          |
| `blockinfile` | Adds a block of lines (multi-line content)          | Insert code snippets      |
| `replace`     | RegEx-based replace in files                        | Change patterns           |
| `assemble`    | Joins multiple source files into one                | Merge config parts        |
| `fetch`       | Pulls a file from remote → control node             | Download logs, backups    |
Μέσα στο repository:

```
mkdir files
nano files/default_site.html

<html> 
  <head>
    <title>Web-site test</title>
  </head>
  <body>
    <h1>It works!</h1>
  </body>
</html>

```
Πάμε να δημιουργήσουμε μια ιστοσελίδα με τη μέθοδο **copy**
install_apache_create_website.yml
```
---
- name: Install Apache with PHP support on all hosts
  hosts: all
  become: true

  tasks:
    - name: Update APT package index
      apt:
        update_cache: yes

    - name: Install Apache2 and PHP
      apt:
        name:
          - apache2
          - libapache2-mod-php
          - php
        state: latest

    - name: Ensure Apache is running and enabled
      service:
        name: apache2
        state: started
        enabled: yes

    - name: Create test html page
      copy:
        src: default_site.html
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: 0644

```
- Η task έχει τίτλο: **"Create test html page"** — είναι απλώς μια περιγραφή που εμφανίζεται στο output όταν τρέχεις το playbook.
    
- Χρησιμοποιείται το Ansible module **`copy`**, το οποίο αντιγράφει ένα αρχείο **από το control node (τοπικό μηχάνημα)** στον **remote server**.
    
- Το **`src: default_site.html`** δηλώνει το όνομα του αρχείου που υπάρχει στον φάκελο του playbook. Είναι το αρχείο που θα αντιγραφεί.
    
- Το **`dest: /var/www/html/index.html`** δηλώνει τη διαδρομή στο απομακρυσμένο μηχάνημα όπου θα τοποθετηθεί το αρχείο. Σε αυτήν τη θέση, ο Apache ψάχνει το αρχικό `index.html`.
    
- Με **`owner: root`** και **`group: root`**, ορίζεις ότι το αρχείο θα ανήκει στον χρήστη και στην ομάδα `root` στον server.
    
- Το **`mode: 0644`** καθορίζει τα δικαιώματα του αρχείου:
    
    - Ο ιδιοκτήτης (root) μπορεί να το διαβάζει και να το γράφει.
        
    - Όλοι οι άλλοι χρήστες μπορούν μόνο να το διαβάζουν.

Άλλο παράδειγμα 

```
- hosts: workstations
  become: true
  tasks: 

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

