---
title: Sysprep
description: Sysprep pour préparer une image à déployer sur d'autres postes.
published: 1
date: 2022-05-07T18:00:23.039Z
tags: windows, sysprep, deployment, déploiement
editor: markdown
dateCreated: 2022-05-07T18:00:18.520Z
---

# Sysprep

## Lancer sysprep par ligne de commande

D'abord éxécuter le cmd en administrateur,
puis

`cd c:/windows/system32/sysprep`
`sysprep /oobe /generalize /shutdown`
- **/generalize**: enlever les configurations spécifiques
- **/oobe** ou **/audit**: mettre en mode oobe OU audit pour installer des logiciels etc
- **/shutdown**: éteindre la machine après le programme
`
## Fichier unattend basique pour supprimer OOBE
Semble ne pas fonctionner, à tester
```xml
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
  <settings pass="oobeSystem">
    <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <OOBE>
        <SkipMachineOOBE>true</SkipMachineOOBE>
        <SkipUserOOBE>true</SkipUserOOBE>
      </OOBE>
    </component>
  </settings>
</unattend>

```

source: https://openclassrooms.com/fr/courses/1733521-installez-et-deployez-windows-10/7379469-generalisez-une-image