---
title: Carte Réseau
description: Modifier les cartes réseaux sous Debian
published: 1
date: 2022-05-05T18:08:41.643Z
tags: linux, carte réseau, network, interfaces
editor: markdown
dateCreated: 2022-05-05T15:47:23.099Z
---

# Mode de fonctionnement
Afin de modifier les paramètres des cartes réseaux il faut:
- modifier le fichier de configuration lié:  `sudo nano /etc/network/interfaces`
- puis redémarrer le service: `service networking restart`
Il peut arriver que cela ne suffise pas, dans ce cas redémarrer la machine: `reboot`

# Statique
Définir une IP statique sur une carte: 

```bash
#Pour les 2 lignes de codes suivantes, possibilité de spécifier plusieurs interfaces en les espacant d'un espace
auto lo ens192 # Démarrer l'interface ens192 lors du démarrage du système

allow-hotplug ens192 #Démarrer l'interface ens192 à chaud

#Définir la configuration
iface ens192 inet static
    address 192.168.36.10 # adresse de la machine
    netmask 255.255.255.0 # masque de sous réseau
    gateway 192.168.36.253 # passerelle
```
    
# DHCP
Définir une IP via serveur DHCP sur une carte:
```bash
#Pour les 2 lignes de codes suivantes, possibilité de spécifier plusieurs interfaces en les espacant d'un espace
auto lo ens192 # Démarrer l'interface ens192 lors du démarrage du système

allow-hotplug ens192 #Démarrer l'interface ens192 à chaud

#Définir la configuration
iface ens192 inet dhcp
```

# DNS
- Pour modifier le DNS:  `nano /etc/resolv.conf`
```bash
nameserver 192.168.44.10
nameserver 9.9.9.9
```


- Autre possibilité, rajouter cette ligne dans la configuration d'une carte réseau: 
```bash
iface ens192 inet static
    dns-nameservers 192.168.44.10 9.9.9.9 # serveurs DNS, par ordre de priorité
```
# Vérfication
Affichage de l'adresse IP actuellement sur les cartes:`ip addr`
Exemple: 
