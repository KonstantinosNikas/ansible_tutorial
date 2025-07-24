## Τι είναι τα **Ansible Templates**;

### Ορισμός:

Τα **templates** είναι αρχεία (συνήθως config αρχεία) που περιέχουν **Jinja2 μεταβλητές**, όπως `{{ my_variable }}`, και χρησιμοποιούνται για να δημιουργήσεις δυναμικά τελικά αρχεία για κάθε host με βάση τις αντίστοιχες μεταβλητές.

### Κατάληξη:

Έχουν κατάληξη `.j2` (π.χ. `sshd_config_ubuntu.j2`)

### Χρησιμότητα:

- Χρήσιμα όταν έχεις **παρόμοια config αρχεία** με μικρές διαφορές ανά host.
    
- Αντί να διαχειρίζεσαι πολλά ξεχωριστά αρχεία, έχεις **ένα template** και εισάγεις τις **μεταβλητές μέσω `host_vars`**.

**Δημιουργία Φακέλου `templates/` στο ρόλο `base`**
```
cd roles/base
mkdir templates
```

**Αντιγραφή αρχείου `sshd_config` από server → `sshd_config_ubuntu.j2`**
```
scp user@ubuntu-server:/etc/ssh/sshd_config roles/base/templates/sshd_config_ubuntu.j2

```
**Σημαντικό**: Κάθε λειτουργικό σύστημα μπορεί να έχει διαφορετική μορφή `sshd_config`. Γι' αυτό το Ubuntu template δεν πρέπει να χρησιμοποιείται σε CentOS.

**Επεξεργασία του `.j2` template**
Μέσα στο αρχείο `sshd_config_ubuntu.j2`, **πρόσθεσε ή αντικατάστησε** τη γραμμή:
```
AllowUsers {{ ssh_users }}
PasswordAuthentication {{ passwd_auth }}
```
Το Ansible θα αντικαταστήσει τις μεταβλητές με τις τιμές από `host_vars`.

**Προσθήκη των μεταβλητών στα `host_vars` στο αρχείο `host_vars/web01.yml`**
```
ssh_users: "jay simone"
ssh_template_file: sshd_config_ubuntu.j2
passwd_auth: no
```
Έτσι, κάθε host μπορεί να χρησιμοποιήσει:
- διαφορετικούς χρήστες
    
- διαφορετικό template
    
- επιλογή για password authentication

**Επεξεργασία του `tasks/main.yml` στον `base` ρόλο**
```
- name: openssh | generate sshd_config file from template
  tags: ssh
  template:
    src: "{{ ssh_template_file }}"
    dest: /etc/ssh/sshd_config
    owner: root
    group: root
    mode: 0644
  notify: restart_sshd
```
Αυτό το task:
- παίρνει το σωστό `.j2` αρχείο (ανάλογα με το `host_vars`)
    
- δημιουργεί το `/etc/ssh/sshd_config`
    
- ενημερώνει το Ansible να **τρέξει handler** για restart του sshd

**Δημιουργία του Handler**
Δημιούργησε τον φάκελο:
```
mkdir roles/base/handlers
cd roles/base/handlers
```
Και στο `main.yml` βάλε:
```
- name: restart_sshd
  service:
    name: sshd
    state: restarted
```
Αν το αρχείο `sshd_config` άλλαξε, το sshd θα επανεκκινηθεί **στο τέλος του playbook**.

**Εκτέλεση Playbook**
## Τι Κερδίζεις με Αυτή τη Διαδικασία;

| Πλεονέκτημα              | Περιγραφή                                                                        |
| ------------------------ | -------------------------------------------------------------------------------- |
| Δυναμική Παραμετροποίηση | Κάθε host μπορεί να έχει διαφορετικό `AllowUsers`, `PasswordAuthentication`κ.λπ. |
| Επαναχρησιμοποίηση       | Δεν χρειάζεσαι 100 config αρχεία – μόνο templates + μεταβλητές                   |
| Ευελιξία                 | Κάθε λειτουργικό σύστημα ή host μπορεί να χρησιμοποιεί το δικό του `.j2`         |
| Καθαρότητα               | Χωρίζεις λογική (Ansible) από περιεχόμενο (templates)                            |
