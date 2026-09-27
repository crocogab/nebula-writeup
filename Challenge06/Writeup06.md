# Challenge 06

Les instructions pour ce challenge sont peu détaillées :
```
The flag06 account credentials came from a legacy unix system.
To do this level, log in as the level06 account with the password level06. Files for this level can be found in /home/flag06.
```

Avant de continuer plus j'essaye de comprendre ce qu'est 'legacy unix system'.

### Legacy unix system

Legacy unix system désigne les façons de faire anciennes encore présentes pour compatibilité mais qui entrainent des pratiques dangereuses qu'on utiliserait plus aujourd'hui.

### Exploitation

Je commence par regarder ce qu'il y a dans le répertoire `/home/flag06` :
```bash
level06@nebula:/home/flag06$ ls -la
total 5
drwxr-x--- 2 flag06 level06   66 2011-11-20 20:51 .
drwxr-xr-x 1 root   root     100 2012-08-27 07:18 ..
-rw-r--r-- 1 flag06 flag06   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag06 flag06  3353 2011-05-18 02:54 .bashrc
-rw-r--r-- 1 flag06 flag06   675 2011-05-18 02:54 .profile
```

Je ne trouve rien de notable ou qui pourrait nous aider.

Comme il faut se connecter avec un mot de passe (il n'y a pas de SSH) et on sait qu'il faut trouver le mot de passe.

Je regarde alors le contenu de `/etc/passwd` :
```bash
level06@nebula:/home/flag06$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
[...]
flag05:x:994:994::/home/flag05:/bin/sh
level06:x:1007:1007::/home/level06:/bin/sh
flag06:ueqwOCnSGdsuM:993:993::/home/flag06:/bin/sh
level07:x:1008:1008::/home/level07:/bin/sh
flag07:x:992:992::/home/flag07:/bin/sh
level08:x:1009:1009::/home/level08:/bin/sh
flag08:x:991:991::/home/flag08:/bin/sh
level09:x:1010:1010::/home/level09:/bin/sh
```

On trouve quelque chose de très étrange : le flag06 est le seul à avoir `ueqwOCnSGdsuM` à la place de `x`.

### Rappel : Structure du fichier `/etc/passwd`

Chaque ligne du fichier correspond à un compte du système.

De plus le format d'une ligne est : `name:password:UID:GID:GECOS:home:shell`.

Découpé champ par champ :
1. `name` : le nom du compte
2. `password` : le mot de passe du compte
3. `UID` : l'identifiant du compte (avec `0`=root)
4. `GID` : l'identifiant du groupe principal
5. `GECOS` : un champ descriptif : purement décoratif
6. `home` : le repertoire personnel ou on atterit à la connexion
7. `shell` : le shell lancé à la connexion (`/usr/bin/nologin` pour interdire le login)

### Solution

Après avoir détaillé la structure du fichier il est clair qu'on a le mot de passe utilisateur hashé.
`flag06:ueqwOCnSGdsuM:993:993::/home/flag06:/bin/sh`

A partir de la il nous suffit de casser le mot de passe avec john.
```bash
level06@nebula:/home/flag06$ grep flag06 /etc/passwd > /tmp/hash.txt
level06@nebula:/home/flag06$ cat /tmp/hash.txt 
flag06:ueqwOCnSGdsuM:993:993::/home/flag06:/bin/sh
```

On trouve assez rapidement le format sur internet :

`ueqwOCnSGdsuM - Possible algorithms: descrypt, DES (Unix), Traditional DES`

J'utilise ensuite les fonctions de john the ripper pour cracker cela sur mon debian perso.

```bash
/usr/sbin/john --format=descrypt /tmp/hash.txt --show
flag06:hello:993:993::/home/flag06:/bin/sh
1 password hash cracked, 0 left
```

Le mot de passe est donc `hello`.

```bash
level06@nebula:/home/flag06$ su flag06
Password: 
sh-4.2$ id
uid=993(flag06) gid=993(flag06) groups=993(flag06)
sh-4.2$ /bin/getflag
You have successfully executed getflag on a target account
```

Bingo on s'est bien connecté !

Conclusion : pas mettre de password dans le `/etc/passwd`.