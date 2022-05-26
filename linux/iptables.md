---
title: Iptables
description: Iptables sous Debian pour réaliser du filtrage, NAT/PAT
published: 1
date: 2022-05-20T07:39:45.131Z
tags: debian, filtrage, iptables, linux, nat, nat/pat, pat
editor: markdown
dateCreated: 2022-05-09T06:06:22.461Z
---

# L'outil
Iptables est un paquet permettant d'effectuer du filtrage et du nat/pat sur un routeur.  
Déprécié depuis Debian 10 (Buster), il est remplacé par [nftables](https://wiki.debian.org/nftables), mais reste utilisable en modifiant certains paramètres.

# Installation
```bash
sudo apt install iptables   
```

# Fonctionnement
L’outil fonctionne avec des listes de règles traitées dans l’ordre. Si une trame satisfait une règle, les suivantes ne seront pas traitées.

# Lister les règles
```bash
iptables -L # lister les règles de la table filter par défaut si aucune table sélectionnée
iptables -t filter -L # lister les règles de la table filter
```
# Ajout de règles
Commande principale de l'outil permettant d'ajouter une règle de filtrage ou nat/pat.

## Utilisation de modules
On peut utiliser des modules additionnels pour la correspondance des paquets.  
Au cours d'une commande, il faut ajouter chaque module un à un avec `-m module_à_ajouter` ou `--match module_à_ajouter`.  
Certains modules sont déjà présents et installés sur la machine et donc utilisable après ajout dans la commmande.
```bash
iptables -t filter -m multiport ... # Ajout du module multiport pour spécifier plusieurs ports qui ne sont pas dans une plage dans une commande
```

## Sélection de la table
`-t` : spécifier la table visé. Valeurs possibles: **nat**, **filter**, **mangle**  
*Optionnel: la table visée est définie par défaut en fonction de la règle. Si la règle est ambigue, il peut être utile de la préciser.*

## Position de la règle dans la table de filtrage  
`-A` : ajouter la/les règle(s) à la fin de la chaîne visée  
OU   
`-I 5` : ajouter la/les règle(s) à la position spécifiée, par défaut 1, 1 = au début de la chaîne

## Ports
`--dport 80` : Port de **destination** de la trame.  
`--sport 1024` : Port **source**  de la trame

Spécifier une plage de ports: `80:150` signifie du port 80 au port 150 inclus.

### Module multiport
Le module multiport permet de spécifier plusieurs ports non contigus dans une commande.  
**15 ports max** par commande. Espacer par des virgules.  
`--dports 80,89,512` : Port de **destination** de la trame.   
`--sports 58,60,158` : Port **source**  de la trame 
`-p tcp` : tcp/udp, *définir le protcole active le module multiport implicitement*  
`-m multiport` : activer le module directement

Exemple:
```bash
iptables -A INPUT -p tcp -i eth0 -m multiport --dports 80,443,20,21 -j ACCEPT #accepter les ports 80 443 20 et 21
```

# Suppression de règles
```bash
iptables -D
```
# Usage commun
## Autoriser accès à un serveur web
```bash
# HTTP: port 80
iptables -t filter -A FORWARD -d 192.168.100.5/32 -p tcp --dport 80 # tout hôte -> serveur web
iptables -t filter -A FORWARD -s 192.168.100.5/32 -p tcp --sport 80 # tout hôte <- serveur web

# HTTPS: port 443
iptables -t filter -A FORWARD -d 192.168.100.5/32 -p tcp --dport 443 # tout hôte -> serveur web
iptables -t filter -A FORWARD -s 192.168.100.5/32 -p tcp --sport 443 # tout hôte <- serveur web
```

## Autoriser accès à un serveur DHCP via agent relais
L'agent relais transmettant les requêtes de diffusion MAC aux réseaux concernés, c'est donc des règles INPUT/OUTPUT et non FORWARD qu'il faut utiliser.

Différentes sources ne s'accordent pas à dire si OUTPUT et INPUT sont tous deux obligatoires. De mes tests, en policy DROP, il faut faire les règles INPUT et OUTPUT. [D'autres](https://serverfault.com/questions/191390/iptables-and-dhcp-questions#comment-168106) le disent aussi.

```bash
# DHCP: port 67 et 68
```

## Autoriser accès à un serveur FTP¨
Il existe 2 modes de fonctionnement pour les serveurs FTP: actif / passif.
D'après mes tests et recherches, le seul moyen que j'ai trouvé pour faire fonctionner le mode actif (port 20), est d'utiliser des règles INPUT / OUTPUT et de faire du NAT/PAT sur le routeur.

# iptables-save
Permet de sauvegarder la configuration actuelle dans un fichier.
```bash
iptables-save > /chemin/vers/le/fichier.save #sauvegarder toutes les tables
```
Arguments de la commande:

`-c` : sauvegarder  les compteurs d'octets et de paquets, si pas spécifié par défaut non

`-t` : spécifier la table à sauvegarder, par défaut toutes, valeurs possibles: filter, nat, mangle

# iptables-restore
Permet de restaurer une configuration à partir d'un fichier précédemment généré (sur ce routeur ou un autre, ou à la main)
```bash
iptables-restore < /chemin/vers/le/fichier.save #restaurer toutes les tables
```

Arguments de la commande: identique à iptables-save mais dans le contexte de restauration.



# Exemple
Article détaillé sur la mise en place de filtrage dans un réseau d'entreprise: [AP SIO2 - DMZ Boulard](https://clementgentil.fr/ap-sio2-mise-en-place-de-dmz-dans-le-reseau-boulard/)

# Sources
[Wiki Debian](https://wiki.debian.org/iptables)  
[Man d'Iptables (documentation officielle)](http://www.delafond.org/traducmanfr/man/man8/iptables.8.html)

http://www.devops-blog.net/iptables/iptables-rules-for-nat-with-ftp-active-passive  