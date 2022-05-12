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
L'outil fonctionne avec des listes de règles traitées dans l'ordre. Si une trame satisfait une règle, les suivantes ne seront pas traitées.

# Commandes

## iptables-save
Permet de sauvegarder la configuration actuelle dans un fichier.
```bash
iptables-save > /chemin/vers/le/fichier.save #sauvegarder toutes les tables
```
Arguments de la commande:

`-c` : sauvegarder  les compteurs d'octets et de paquets, si pas spécifié par défaut non

`-t` : spécifier la table à sauvegarder, par défaut toutes, valeurs possibles: filter, nat, mangle



# Exemple
Article détaillé sur la mise en place de filtrage dans un réseau d'entreprise: [AP SIO2 - DMZ Boulard](https://clementgentil.fr/ap-sio2-mise-en-place-de-dmz-dans-le-reseau-boulard/)
