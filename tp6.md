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
1 - 
