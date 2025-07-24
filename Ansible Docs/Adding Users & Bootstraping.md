user_management.yml
```
- hosts: all
  become: true
  tasks:

  - name: create kostas user
    tags: always
    user: 
      name: kostas
      groups: root

  - name: add ssh key for kostas
    tags: always
authorized_key:
  user: kostas
  key: "..."  

  - name: add sudoers file for kostas
    tags: always
    copy:
  src: sudoer_kostas
  dest: /etc/sudoers.d/kostas
  owner: root
  group: root
  mode: 0440
```
Τρέχουμε 
`ansible-playbook user_management.yml`

Με το παραπάνω τρόπο 
- **Δημιουργήσαμε έναν νέο χρήστη με όνομα `kostas`**
- **Προσθέσαμε SSH public key για τον χρήστη `kostas`**
- **Δώσαμε sudo δικαιώματα στον `kostas` χωρίς να πειράξουμε το `/etc/sudoers` απευθείας**
