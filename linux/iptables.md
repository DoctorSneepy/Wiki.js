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

# Politique de filtrage
Définir la politique de filtrage pour les champs `INPUT`, `OUTPUT` et `FORWARD`  
Une politique est le comportement par défaut à adopter pour toutes trames ne correspondant pas aux règles.

```bash
iptables -P INPUT DROP
```

## Valeurs possibles:

`ACCEPT` : Accepter toutes les trames sauf règles  
`DROP`:  Refuser toutes les trames sauf règles, sans informer la source  
`REJECT` :  Refuser toutes les trames sauf règles, en informer la source  

Les 3 tables ne sont pas systématiquement de la même politique. *On peut par exemple avoir une table FORWARD en ACCEPT et les tables INPUT et OUTPUT en DROP.*

## Exemples
```bash
# TOUT ACCEPTER PAR DEFAUT 
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT

# TOUT REFUSER PAR DEFAUT
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT DROP
sudo iptables -P FORWARD DROP
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

## Protocole
Définir le(s) protocole(s) auxquelles s'appliquent la règle: `-p tcp`  
Valeurs possibles: `tcp, udp, udplite, icmp, icmpv6, esp, ah, sctp, mh, all`

Par défaut tous : 0 / all
Spécifier ! avant signifie l'inverse. !tcp veut dire tout sauf tcp.

## Ports
Les ports ne sont utilisés qu'avec les protocoles TCP et UDP.  
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

## IP
`-d 192.168.1.0/2` : Adresse/Réseau IP de **destination** de la trame.  
`-s 10.0.0.7/32` : Adresse/Réseau IP **source**  de la trame

Si pas de masque précisé, par défaut /32 = IP précise.  
Si pas d'adresse IP source et / ou destination précisé, par défaut toutes.

# Suppression de règles
```bash
iptables -D
```
# Usage commun
## Autoriser requête ping
```bash
    # Pare-feu = 192.168.36.253 pour l'exemple
# Ping le parefeu
iptables -t filter -A INPUT -d 192.168.36.253 -p icmp -j ACCEPT # tout hôte -> pare-feu
iptables -t filter -A OUTPUT -s 192.168.36.253 -p icmp -j ACCEPT # tout hôte <- pare-feu

# Ping toute machine derrière le pare-feu sur le réseau 192.168.36.0/24
iptables -t filter -A FORWARD -d 192.168.36.0/24 -p icmp -j ACCEPT
iptables -t filter -A FORWARD -s 192.168.36.0/24 -p icmp -j ACCEPT 
```

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