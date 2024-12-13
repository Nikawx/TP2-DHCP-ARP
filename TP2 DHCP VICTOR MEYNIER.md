# TP2 VICTOR MEYNIER 

# SETUP

## NUAGE NAT
## ROCKY LINUX ROUTER INTERNET
## IOU
## ROCKY LINUX DHCP
## VPCS CLIENT

Le premier serveur ROCKY récupère l'internet via 2 cartes réseaux, une carte pour la NAT et l'autre pour la connection avec l'IOU.

La première machine, le router-tp2 à comme ip :

```
- 192.168.122.172/24 -> NAT
- 10.2.1.254/24 -> RESEAU
```

le ping 8.8.8.8 est effectué.
```
64 bytes from 8.8.8.8 icmp_seq=1 ttl=113 time 17.5 ms
```

# Modification du firewall

```
sudo firewall-cmd --add-masquerade
sudo firewall-cmd --add-masquerade --permanent
```

# Configuration de node1.tp2

IP fixe de la machine :

```
ip 10.2.1.1 255.255.255.0 10.2.1.254
Checking for duplicate address...
PC1 : 10.2.1.1 255.255.255.0 gateway 10.2.1.254
```

Ping Pong des deux machines :

```
PC1> ping 10.2.1.254
84 bytes from 10.2.1.254 icmp_seq=1 ttl=64 time=16.037 ms

[nikawx@localhost ~]$ ping 10.2.1.1
64 bytes from 10.2.1.1: icmp_seq=1 ttl=64 time=3.44 ms
```
PS : La commande traceroute ne marche pas,

     -bash: traceroute: command not found
     même après un sudo dnf install traceroute

# Afficher la CAM Table du Switch IOU

```
IOU5# show mac address-table

Vlan    Mac Address           Type      Ports
1       0050.7966.6800      DYNAMIC      Et0/1
1       0800.278b.2f6c      DYNAMIC      Et0/3

```

# Install et conf du serveur DHCP sur dhcp.tp2

Installation d'un DHCP sur Rocky Linux

Je me suis mis sur une ip fixe via "sudo nmtui"

 - IP : 10.2.1.253/24 

sudo systemctl restart NetworkManager

[nikawx@localhost ~]$ ping 10.2.1.254
64 bytes from 10.2.1.254: icmp_seq=1 ttl=64 time=9.67 ms

# Test du DHCP

```
sudo nmcli con del enp0s3

config dhcpd.conf avec le BONUS DNS (1.1.1.1):

default-lease-time 600;
max-lease-time 7200;
authoritative;
subnet 10.2.1.0 netmask 255.255.255.0 {
    range 10.2.1.2 10.2.1.50;
    option domain-name-servers 8.8.8.8, 8.8.4.4, 1.1.1.1;
    option broadcast-address 10.2.1.255;
    option routers 10.2.1.254;
}


```

Une fois sur la machine client nous pouvons établir la commande "dhcp" pour demander qu'elles addresses sont disponibles et donc en obtenir une.

PC1> DORA IP 10.2.1.2/24 GW 10.2.1.254

PC1> ping efrei.fr
84 bytes from 51.255.68.208 icmp_seq=1 ttl=53 time=31.922 ms

Nous obtenons une addresse ip dans le réseau et un accès INTERNET !


# Wireshark it !

Je te met sur le git DORA.png

# Afficher la table ARP de router.tp2

J'ai donc utilisé ip neigh, 

10.2.1.253 dev enp0s8 lladdr 08:00:27:dd:06:c3 REACHABLE
192.168.122.1 dev enp0s3 lladdr 52:54:00:23:04:c2 DELAY
10.2.1.2 dev enps0s8 lladdr 00:50:79:66:68:00 STALE
10.2.1.1 dev enps0s8 lladdr 00:50:79:66:68:00 STALE

# Capturez l'échange ARP avec Wireshark

Je te met tout sur le GIT ARP.png

# ARP poisoning

# Envoyer une trame ARP arbitraire

```
sudo arping -c 1 -s 10.2.1.254 10.2.1.2
ARPING 10.2.1.2 from 10.2.1.254 enp0s8
Unicast reply from 10.2.1.2 [00:50:79:66:68:00] 5.063ms
Unicast reply from 10.2.1.2 [00:50:79:66:68:00] 7.758ms
Sent 1 probes (1 broadcast(s))
Received 2 response(s)

```
# Mettre en place un ARP MITM

Je viens d'ajouter une machine Kali Linux au réseau je l'ai branché au IOU, j'ai obtenu directement une addresse ip en 10.2.1.3 qui ping le réseau et qui ping 8.8.8.8 ! trop facile 

Sur la Kali Linux je décide donc d'utiliser Ethercat qui est un tool très utile est efficace pour le ARP Poisoning je l'utilise souvent.

Je me suis mis un eth0 j'ai lancé un scan des hosts j'ai pu récupérer ça :

IP Address          MAC Address
10.2.1.2            00:50:79:66:68:00
10.2.1.253          08:00:27:DD:06:C3
10.2.1.254          08:00:27:8B:2F:6C

J'ajoute 10.2.1.2 00:50:79:66:68:00 comme Target 1 (Node1)
J'ajoute 10.2.1.254 08:00:27:8B:2F:6C comme Target 2 (Router1)

Après tout cela j'ai fais un ping 1.1.1.1 avec la machine victime et dans le wireshark j'ai pu capturer la requete et carrément la cerise sur le gateau une alerte : 

"Duplicate IP address detected for 10.2.1.2 also in use by 00:50:79:66:68:00"












