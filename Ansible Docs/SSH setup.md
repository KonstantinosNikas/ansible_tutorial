#### SSH Key Managment 
Για να δούμε τα κλειδιά που έχουμε αποθηκευμένα:
```
ls -la .ssh 
```
#### SSH Key Generate 
Για να δημιουργήσουμε ένα κλειδί:
```
ssh-keygen -t ed25519 -C "nameofkey"
```
Το output που πρέπει να δούμε:
```
Generating public/private ed25519 key pair.

Enter file in which to save the key (/Users/kostasnikas/.ssh/id_ed25519): 

Enter passphrase for "/Users/kostasnikas/.ssh/id_ed25519" (empty for no passphrase): 

Enter same passphrase again: 

Your identification has been saved in /Users/kostasnikas/.ssh/id_ed25519

Your public key has been saved in /Users/kostasnikas/.ssh/id_ed25519.pub

The key fingerprint is:

SHA256:vvtI/hsWZgkE0ybFrhRVfOjQoxhsjQ4bDhF2kXU3ao4 nameofkey

The key's randomart image is:

+--[ED25519 256]--+

|  +oo=+O=++.     |

| ...+ B+*o=..    |

|   o * B++ o     |

|    o ++o...     |

|     .E.S =      |

|      .. o .     |

|        o o      |

|       o + .     |

|        =++.     |

+----[SHA256]-----+
```
#### Adding a SSH Key 
Για να στείλουμε το public key σε ένα host που θέλουμε χρησιμοποιούμε την εντολή:
```
ssh-copy-id -i ~/.ssh/id_ed25519.pub <name@ip>
```

#### Creating an Ansible Key 
Χρειάζεται να δημιουργήσουμε ενα κλειδί μόνο για την **Ansible**
```
ssh-keygen -t ed25519 -C "ansible"
```
Στο "Enter file in which to save the key (/Users/kostasnikas/.ssh/id_ed25519):" βάζουμε:
```
/home/pathname/.ssh/<nameofkey>
```
#### Adding Ansible SSH Key 
```
ssh-copy-id -i ~/.ssh/<nameofkey>.pub <ip>
```

#### Use specific SSH Key
Αν θέλω να επιλέξω συγκεκριμένο key για είσοδο
```
ssh -i ~/.ssh/<nameofkey> <ip>
```
#### SSH Agent Cheat
Για να μη χρειάζεται να γράφω συνέχεια το passphrase οταν κάνω ssh: 
```
eval $(ssh-agent)
ssh-add
alias ssha='eval $(ssh-agent) && ssh add'
ssha

```
και δίνω passhphrase  που θέλω να συγκρατήσει, τέλος προσαρμόζω κατάλληλα το .bashrc, βρίσκω κενή γραμμή και προσθέτω
```
nano .bashrc

# ssh-agent
alias ssha='eval $(ssh-agent) && ssh add'

```
