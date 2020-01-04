# Write up de la box Craft 

## Intro' 

Juste pour ceux qui ne savent pas HackTheBox est une plateforme de CTF (ou box) ou on se connecte à un VPN pour "attaquer" des box sur ce reseau pour avoir un shell en user puis en root dessus. Les box sont retiré au bout d'un moment, il n'y a que 20 box active (il y a une nouvelle box chaque samedi). On peut acceder au 2 dernière box retiré mais aussi aux autres si ont a le VIP (10 $/moi).
Voila !
Sinon un petit "respect" sur [mon hackthebox](https://www.hackthebox.eu/home/users/profile/200229) serait sympa ! 
Et je vous rappelle l'existence de  [mon serveur discord](https://discord.gg/2bwhtP7)
C'était pour la pub ! Maintenant passons au choses serieuses :) 

## Enumération du début

On va commencer par rejouter l'ip de la box a /etc/host (Linux):
```
root@DESKTOP-8OBCBE1:/home/quasar# echo "10.10.10.110 craft.htb" >> /etc/hosts
root@DESKTOP-8OBCBE1:/home/quasar#
```
(oui je suis sous windows et j'use le WSL, et alors mdr ? ) 
Ou plus simplement si vous êtes sous windows 

```
PS C:\Windows\system32\drivers\etc> bash -c "echo '10.10.10.110 craft.htb' >> hosts"
PS C:\Windows\system32\drivers\etc> cat hosts
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
#       127.0.0.1       localhost
#       ::1             localhost
10.10.10.110 craft.htb
PS C:\Windows\system32\drivers\etc>
```

Tres bien ! scannons les ports avec nmap maintenant:
```
C:\Users\quasa>nmap -A craft.htb
WARNING: Could not import all necessary Npcap functions. You may need to upgrade to the latest version from https://npcap.org. Resorting to connect() mode -- Nmap may not function completely
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-04 07:53 Changement de date
Nmap scan report for craft.htb (10.10.10.110)
Host is up (0.11s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.4p1 Debian 10+deb9u5 (protocol 2.0)
| ssh-hostkey:
|   2048 bd:e7:6c:22:81:7a:db:3e:c0:f0:73:1d:f3:af:77:65 (RSA)
|   256 82:b5:f9:d1:95:3b:6d:80:0f:35:91:86:2d:b3:d7:66 (ECDSA)
|_  256 28:3b:26:18:ec:df:b3:36:85:9c:27:54:8d:8c:e1:33 (ED25519)
443/tcp open  ssl/http nginx 1.15.8
|_http-server-header: nginx/1.15.8
|_http-title: About
| ssl-cert: Subject: commonName=craft.htb/organizationName=Craft/stateOrProvinceName=NY/countryName=US
| Not valid before: 2019-02-06T02:25:47
|_Not valid after:  2020-06-20T02:25:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn:
|_  http/1.1
| tls-nextprotoneg:
|_  http/1.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 103.13 seconds

C:\Users\quasa>
```

Comme vous remarquez le port 443 est ouvert avec du https, regardons cela.
![Image](crafthtb.png)

En regardans le code source de la page (ou juste en haut à droite) on trouve:
```html
<ul class="nav navbar-nav pull-right">
            <li><a href="https://api.craft.htb/api/">API</a></li>
            <li><a href="https://gogs.craft.htb/"><img border="0" alt="Git" src="/static/img/Git-Icon-Black.png" width="20" height="20"></a></li>
          </ul>

```

Rajoutons donc les sous domaine api et gogs à c:/windows/system32/drivers/etc/hosts:
```
10.10.10.110 craft.htb api.craft.htb gogs.craft.htb
```
Regardons maintenant l'api: 
![Image](crafthtb2.png)

En regardant ce qu'on peux faire, le seul truc que je trouve interessant est:
![Image](crafthtb3.png)

Nous avons pas de credentials continuons l'énumération sur le gogs:
![Image](crafthtb4.png)
Je me suis un peu documenté sur gogs et chercher s'il n'y a pas de vulnerabilité mais apparament non, j'ai donc continuer l'énumération en allant sur Explorer pus en cliquand sur CRaft/Craft API ont tombe sur ça:

![image](crafthtb5)
