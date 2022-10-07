# __TP 6 - Services réseau__

_Exercice 1. Adressage IP (rappels)_

Numéro du sous réseau | Adresse de sous réseau | Adresse de broadcast | Adresse de la première machine | Adresse de la dernière machine 
------------ | ------------- | ------------ | ------------- | ------------ | 
 1 | 172.16.0.0 | 172.16.0.63 | 172.16.0.1 | 172.16.0.62 
2 | 172.16.0.64 | 172.16.0.127 | 172.16.0.65 | 172.16.0.126
3 | 172.16.0.128 | 172.16.0.191 | 172.16.0.129 | 172.16.0.190
4 | 172.16.0.192 | 172.16.0.255 | 172.16.0.193 | 172.16.0.254
5 | 172.16.1.0 | 172.16.1.1 | 172.16.1.62 | 172.16.1.63
6 | 172.16.1.64 | 172.16.1.127 | 172.16.1.65 | 172.16.1.126
7 | 172.16.1.128 | 172.16.1.159 | 172.16.1.129 | 172.16.1.158

_Exercice 2. Préparation de l’environnement_ 

2 - L'interface "lo" correspond à loop back, la boucle locale.   
3 -
```
User@localhost:~$ sudo apt-get remove cloud-init
```
4 -
```
sudo hostnamectl set-hostname "NOM"
````
_Exercice 3. Installation du serveur DHCP_   
1 - J'installe les paquets via 
```
sudo apt install isc-dhcp-server
```
2 - Pour modifier l'adresse ip de l'interface réseau il faut se rendre dans le fichier "/etc/netplan/50-cloud-init.yaml"
```
network:
  ethernets:
      ens192:
          dhcp4: true
          match:
              macaddress: 00:50:56:89:3a:1d
          set-name: ens192
       ens224:
           dhcp4: true
           addresses: [192.168.100.1/24]
           nameservers:
               addresses: [8.8.8.8, 1.1.1.1]
  version: 2
```
3 - J'ai fais une copie de "/etc/dhcp/dhcpd.conf" nommé "dhcpd.conf.bak":
```
sudo cp dhcpd.conf dhcpd.conf.bak
```
Les deux premières lignes correspondent au temps par défaut et au temps maximal du bail en seconde : 
```
default-lease-time 120;
max-lease-time 600;
```
4 - Il faut modifier le fichier "/etc/default/isc-dhcp-server".
```
INTERFACESv4="ens224"
INTERFACESv6="ens224"
```

5 - Je valide la configuration du serveur avec un "dhcpd -t" puis je le redémarre avec "systemctl restart isc-dhcp-server".  

6 - Pour désinstaller "cloud-init" je fais un nouveau :
```
User@localhost:~$ sudo apt-get remove cloud-init
```
Puis pour renomer la machine en "client.tpadmin.local"
```
sudo hostnamectl set-hostname client.tpadmin.local
```
Et pour finir je modifie aussi le nom de la machine dans le /etc/hosts.  

7 - 
``` 
DHCPREQUEST for 192.168.100.100 from 00:50:56:89:b3:60 via ens224
DHCPACK on 192.168.100.100 to 00:50:56:89:b3:60 via ens224
DHCPDISCOVER from 00:50:56:89:ee:94 via ens224
DHCPOFFER on 192.168.100.101 to 00:50:56:89:ee:94 via ens224
```
- DHCP REQUEST = La demande d'adresse ip du client au serveur
- DHC PACK = La confirmation de la récepetion de la demande du serveur
- DHCP DISCOVER = Le serveur trouve une adresse disponible à donner au client
- DHC POFFER = La réponse du serveur au client, il lui envoie son adresse IP  

8 - "/var/lib/dhcp/dhcpd.leases" contient les demandes de bails des clients, quant à elle, la commande "dhcp-lease-list" liste les bails avec leursdates d'expirations.  

9 - Je ping le client depuis le serveur avec un "ping 192.168.100.100"   
```
PING 192.168.100.100 (192.168.100.100) 56(84) bytes of data.
64 bytes from 192.168.100.100: icmp_seq=1 ttl=64 time=0.248 ms
64 bytes from 192.168.100.100: icmp_seq=2 ttl=64 time=0.271 ms
64 bytes from 192.168.100.100: icmp_seq=3 ttl=64 time=0.236 ms
64 bytes from 192.168.100.100: icmp_seq=4 ttl=64 time=0.245 ms
^C
```
10 - Après la modification faite, l'adresse de l'interface à bien changé.

# _Exerice 4.  Donner un accès à Internet au client_

1 - Pour activer l'ip forwarding sur le serveur je dois décommenter la partie suivant dans "/etc/sysctl.conf" :
```
net.ipv4.ip_forward=1
```
Puis ensuite un "sudo sysctl -p /etc/sysctl.conf" pour mettre à jour le changement sur le serveur. Pour vérifier que la modification est bien active je fais un "sysctl net.ipv4.ip_forward" et je vérifie qu'on me renvoie bien un :
```
net.ipv4.ip_forward = 1
```

2 - J'autorise la traduction d'adresse source grâce à :
```
sudo iptables --table nat --append POSTROUTING --out-interface ens192 -j MASQUERADE
```
Je vérifie ensuite que je peux bien ping 1.1.1.1 depuis le client :
```
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=53 time=9.40 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=53 time=8.94 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=53 time=8.84 ms
```
# _Exercice 5. Installation du serveur DNS_

1 - J'installe bind9 avec un "apt install bind9", je fais ensuite un systemctl status bind9 pour m'assurer qu'il est bien installé.  

2 - Je modifie donc le fichier de configuration de bind9 "/etc/bind/named.conf.options", j'y décommenté la partie du forwarders et j'y ajoute les adresses IP des DNS publics 1.1.1.1 et 8.8.8.8 :
```
forwarders{
1.1.1.1;
8.8.8.8;
};
```
Je redémarre ensuite bind9 pour actualiser la modification apporté sur le serveur.  

3 - Le ping de www.google.fr depuis le client fonctionne : 
```
PING www.google.fr (216.58.209.227) 56(84) bytes of data.
64 bytes from par10s29-in-f3.1e100.net (216.58.209.227): icmp_seq=1 ttl=111 time=8.85 ms
64 bytes from par10s29-in-f3.1e100.net (216.58.209.227): icmp_seq=2 ttl=111 time=8.88 ms
64 bytes from par10s29-in-f3.1e100.net (216.58.209.227): icmp_seq=3 ttl=111 time=8.61 ms
64 bytes from par10s29-in-f3.1e100.net (216.58.209.227): icmp_seq=4 ttl=111 time=8.55 ms
```
4 - Je réutilise lynx installé dans un précédent tp pour naviguer sur une page web avec :
```
lynx fr.wikipedia.fr
```

# _Exercice 6. Configuration du serveur DNS pour la zone tpadmin.local_

1 - Je modifie le fichier "/etc/bin/db.tpadmin.local" en ajoutant : 
```
zone "tpadmin.local" IN {
type master; 
file "/etc/bind/db.tpadmin.local"; 
};
```
2 - Je crée donc une copie de "db.local" pour ensuite la renommer "db.tpadmin.local" pour enfin modifier ce dernier en remplacant les "localhost" en m'assurant de bien mettre l'adresse du serveur : 
```
$TTL    604800
@       IN      SOA     tpadmin.local. root.tpadmin.local. (
                            2           ; Serial
                       604800           ; Refresh 
                        86400           ; Retry   
                      2419200           ; Expire  
                       604800 )         ; Negative Cache TTL 
;
@       IN      NS      tpadmin.local.
@       IN      A       192.168.100.1
@       IN      AAAA    ::1
```
3 - J'ajoute ensuite dans le fichier "named.conf.local" :

```
zone "100.168.192.in-addr.arpa" {
type master;
file "/etc/bind/db.192.168.100";
};
```
Je m'occupe ensuite du fichier "named.conf.local" (Il faut mettre l'@ IP du serveur à l'envers puisqu'il s'agit d'un fichier de configutation de zone inverse) :
```
$TTL    604800
@       IN      SOA     tpadmin.local. root.tpadmin.local. (
                            1           ; Serial
                       604800           ; Refresh 
                        86400           ; Retry   
                      2419200           ; Expire  
                       604800 )         ; Negative Cache TTL 
;
@       IN      NS      tpadmin.local.
1.100.168.192   IN      PTR     tpadmin.local.l
```
4 - Via l'utilisation des commandes "named-checkconf" et " named-checkzone" je regarde si les modifications ont bien fonctionnées : 

```
User@tpadmin:/etc/bind$ named-checkconf named.conf.local
User@tpadmin:/etc/bind$ named-checkzone tpadmin.local /etc/bind/db.tpadmin.local
zone tpadmin.local/IN: loaded serial 2
OK
User@tpadmin:/etc/bind$ named-checkzone 100.168.192.in-addr.arpa /etc/bind/db.192.168.100
zone 100.168.192.in-addr.arpa/IN: loaded serial 1
OK
```
5 - Après redémarrage de mon serveur Bind9 les pings fonctionnent :
```
PING serveur.tpadmin.local (192.168.100.1) 56(84) bytes of data.
64 bytes from serveur.tpadmin.local (192.168.100.1): icmp_seq=1 ttl=64 time=0.388 ms
64 bytes from serveur.tpadmin.local (192.168.100.1): icmp_seq=2 ttl=64 time=0.423 ms
64 bytes from serveur.tpadmin.local (192.168.100.1): icmp_seq=3 ttl=64 time=0.196 ms
64 bytes from serveur.tpadmin.local (192.168.100.1): icmp_seq=4 ttl=64 time=0.498 ms
```

