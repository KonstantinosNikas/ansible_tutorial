To see if **Git** is installed in our machine 
```
which git 
```
and we must see something like this 
```
/usr/bin/git
```

- **Δημιουργώ στο github  ένα repository** 
- Πηγαίνω στα Settings --> SSH and GPG Keys --> New SSH Key
και βάζω το .pub key που δημιουργησα για τις ssh συνδέσεις με servers.
- Κατεβάζω με git clone το repository στο workstation μου.
- Πριν αρχίσουμε να εργαζόμαστε στο Repository, θα κάνουμε config τα στοιχεία του χρήστη μας.
```
git config --global user.name "Kostas Nikas"
git config --global user.mail "kostas@nikas.gr" 

cat ~/.gitconfig
```

**Βασικές Εντολές Git**
```
git status

git diff <filename>

git add . or <filename>

git commit -m "some message"

git push origin main
```