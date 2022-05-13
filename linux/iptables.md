---
title: Iptables
description: Iptables sous Debian pour réaliser du filtrage, NAT/PAT
published: 1
date: 2022-05-09T06:06:30.085Z
tags: linux, iptables, filtrage, nat, pat, nat/pat, debian
editor: markdown
dateCreated: 2022-05-09T06:06:22.461Z
---

# L'outil
Iptables est un paquet permettant d'effectuer du filtrage et du nat/pat sur un routeur.

# Installation
```bash
sudo apt install iptables   
```

# Fonctionnement
L’outil fonctionne avec des listes de règles traitées dans l’ordre. Si une trame satisfait une règle, les suivantes ne seront pas traitées.

# Commandes

## iptables
Commande principale de l'outil permettant d'ajouter une règle de filtrage ou nat/pat.

Arguments: 

## Usage commun
### Autoriser accès à un serveur web
```bash
# HTTP: port 80
iptables -t filter -A FORWARD -d 192.168.100.5/32 -p tcp --dport 80 # tout hôte -> serveur web
iptables -t filter -A FORWARD -s 192.168.100.5/32 -p tcp --sport 80 # tout hôte <- serveur web

# HTTPS: port 443
iptables -t filter -A FORWARD -d 192.168.100.5/32 -p tcp --dport 443 # tout hôte -> serveur web
iptables -t filter -A FORWARD -s 192.168.100.5/32 -p tcp --sport 443 # tout hôte <- serveur web
```
### Autoriser accès à un serveur FTP

### Autoriser accès à un serveur DNS

### Autoriser accès à un serveur DHCP via agent relais
L'agent relais transmettant les requêtes de diffusion MAC aux réseaux concernés, c'est donc des règles INPUT/OUTPUT et non FORWARD qu'il faut utiliser.

Différentes sources ne s'accordent pas à dire si OUTPUT et INPUT sont tous deux obligatoires. De mes tests, en policy DROP, il faut faire les règles INPUT et OUTPUT. [D'autres](https://serverfault.com/questions/191390/iptables-and-dhcp-questions#comment-168106) le disent aussi.

```bash
# DHCP: port 67 et 68
iptables -t filter -A INPUT -d 192.168.100.6/32 -p tcp --dport 67:68 --sport 67:68 # tout hôte -> serveur web
```

## iptables-save
Permet de sauvegarder la configuration actuelle dans un fichier.
```bash
iptables-save > /chemin/vers/le/fichier.save #sauvegarder toutes les tables
```
Arguments de la commande:

`-c` : sauvegarder  les compteurs d'octets et de paquets, si pas spécifié par défaut non

`-t` : spécifier la table à sauvegarder, par défaut toutes, valeurs possibles: filter, nat, mangle

## iptables-restore
Permet de restaurer une configuration à partir d'un fichier précédemment généré (sur ce routeur ou un autre, ou à la main)
```bash
iptables-restore < /chemin/vers/le/fichier.save #restaurer toutes les tables
```

Arguments de la commande: identique à iptables-save mais dans le contexte de restauration.


# Exemple
Article détaillé sur la mise en place de filtrage dans un réseau d'entreprise: [AP SIO2 - DMZ Boulard](https://clementgentil.fr/ap-sio2-mise-en-place-de-dmz-dans-le-reseau-boulard/)
