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
```

Comme vous remarquez le port 443 est ouvert avec du https, regardons cela.
![Image](crafthtb.png)
