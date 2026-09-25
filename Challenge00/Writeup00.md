# Challenge 00


Set User ID (`setuid`) : les droits setuid permettent d'executer un executable avec les permissions de la personne qui possède l'executable.

Pour trouver un executable avec le SUID de flag00 : `find / -perm -4000 -type f 2>/dev/null`.

Explication :
- `find` : fonction pour trouver dossier / fichier
- `/` : à partir de quel dossier on regarde
- `-perm` : predicat qui teste les bits de permissions du fichier
- `-4000` : au minimum les bits d'executions sont là

Le bit de suid nous dit donc : *exécute-moi avec les privilèges du propriétaire du fichier, pas de celui qui me lance.*

```bash
level00@nebula:/bin/...$ ./flag00 
Congrats, now run getflag to get your flag!
flag00@nebula:/bin/...$ getflag
You have successfully executed getflag on a target account
```