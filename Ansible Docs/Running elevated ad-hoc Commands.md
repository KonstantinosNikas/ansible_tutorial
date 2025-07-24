Αν θέλω να κάνω update σε όλους τους hosts 
```
ansible all -m apt -a update_cache=true --become --ask-become-pass
```

**Εκτελεί `apt update` σε όλα τα hosts**, δηλαδή ανανεώνει την cache των πακέτων (τα διαθέσιμα updates από τα repos), όπως θα έκανες με:
`sudo apt update`

Αν θέλω να κάνω εγκατάσταση κάποιο λογισμικό για παράδειγμα το Vim μπορώ να τρέξω.
```
ansible all -m apt -a name=vim-nox --become --ask-become-pass
```
**Εγκαθιστά το πακέτο `vim-nox` σε όλα τα remote hosts** μέσω του APT.
```
ansible all -m apt -a "name=vim-nox state=latest" --become --ask-become-pass
```
`-a "name=vim-nox state=latest"`: εγκαθιστά το πακέτο `vim-nox` ή το ενημερώνει στην τελευταία διαθέσιμη έκδοση.
```
ansible all -m apt -a "upgrade=dist" --become --ask-become-pass
```
`-a "upgrade=dist"`: Ζητάει να γίνει `dist-upgrade`, δηλαδή:
- Αναβαθμίζει όλα τα πακέτα στο νεότερο διαθέσιμο.
- Μπορεί να **προσθέσει ή αφαιρέσει εξαρτήσεις** (σε αντίθεση με `upgrade=yes`).
