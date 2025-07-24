Για να κατεβάσουμε την Ansible:
```
sudo apt update 
sudo apt install ansible 
```
Mέσα στο git repository  ποy έχουμε κάνει clone στο workstation δημιουργούμε το αρχείο inventory.
```
nano inventory 
```
Μέσα στο inventory γραφουμε τις IP των hosts π.χ.
```
172.16.250.132
172.16.250.133
172.16.250.134
```
Για να ελέξω τη σύνδεση μεταξύ όλων των hosts χρησιμοποιώντας το κλείδι της ansible που δημιουργήσαμε 
```
ansible all --key-file ~/.ssh/<filenameforansible> -i inventory -m ping 
```
- `ansible`: Εκτελεί την εντολή μέσω του Ansible (εργαλείο αυτοματοποίησης για servers).
    
- `all`: Εφαρμόζει την εντολή σε **όλους** τους κόμβους (servers) που περιγράφονται στο αρχείο "inventory".
    
- `-i inventory`: Δηλώνει το αρχείο **inventory**, δηλαδή το αρχείο που περιέχει τις IP ή ονόματα των servers στους οποίους θέλουμε να συνδεθούμε.
    
- `--key-file ~/.ssh/<filenameforansible>`: Δηλώνει το **ιδιωτικό SSH κλειδί** που θα χρησιμοποιήσει το Ansible για να συνδεθεί με ασφάλεια στους servers.
    
    - Π.χ. `~/.ssh/ansible_key.pem`
        
- `-m ping`: Λέει στο Ansible να εκτελέσει το **module "ping"**, το οποίο ελέγχει απλώς αν μπορεί να συνδεθεί σωστά μέσω SSH στους servers.

Για να αυτοματοποιήσουμε περισσοότερα τα πράγματα μπορούμε να δημιουργήσουμε ενα configuration file.
```
nano ansible.cfg
```
Μέσα στο αρχείο μπορώ να βάλω πολλά entries όπως 
```
[default]
inventory = inventory
remote_user = debian
private_key_file = ~/.ssh/ansible
```
`inventory = inventory`: Χρησιμοποιεί το αρχείο `inventory` (θα πρέπει να υπάρχει στον ίδιο φάκελο).
`private_key_file = ~/.ssh/ansible`: Χρησιμοποιεί το ιδιωτικό κλειδί `~/.ssh/ansible` για SSH σύνδεση.

Τώρα με το αρχείο που δημιουργήσαμε μπορουμε να τρέξουμε 
```
ansible all -m ping 
```
και να έχουμε το ίδιο αποτέλεσμα με την εντολή που τρέξαμε παραπάνω.

```
ansible all -m gather_facts --limit <ip> 
```
`-m gather_facts`: Εκτελεί το module `gather_facts`, το οποίο μαζεύει πληροφορίες για το σύστημα του host.