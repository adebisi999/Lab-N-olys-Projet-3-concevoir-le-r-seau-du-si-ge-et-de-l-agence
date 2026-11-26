# Journal de dépannage – Projet 3

### 1. La webcam IoT n'a pas de port Ethernet
- **Cause** : dans Packet Tracer, les objets IoT sont livrés avec une carte Wi-Fi.
- **Solution** : Config › Advanced › I/O Config › Network Adapter : **PT-IOT-NM-1CFE**.

### 2. Confusion entre les ports G1/0/x et G0/x
- **Cause** : G1/0/x sont les ports du switch 3650 ; un routeur a des ports G0/0/x (ISR4331) ou G0/x (2911).
- **Leçon** : toujours vérifier le nom réel des ports avec `show ip interface brief`.

### 3. `%CDP-4-NATIVE_VLAN_MISMATCH` entre SW-CORE et SW-ACC1
- **Cause** : VLAN natif 999 configuré sur le switch d'accès, mais encore 1 sur SW-CORE.
- **Risque** : un VLAN natif différent des deux côtés peut faire passer du trafic dans le mauvais VLAN (VLAN hopping).
- **Solution** : `switchport trunk native vlan 999` des deux côtés.

### 4. Le PC n'obtient pas d'adresse DHCP (169.254.x.x)
- **Diagnostic** : SVI up/up et pools DHCP présents, mais `show interfaces trunk` montrait « Vlans allowed on trunk : 1-9,11-19,21-49… ».
- **Cause** : les VLAN 10, 20, 50, 60, 70, 80, 90 et 99 avaient été retirés des trunks (`switchport trunk allowed vlan remove`).
- **Solution** : `switchport trunk allowed vlan all` sur les trunks.
- **Leçon** : un trunk « up » ne fait pas forcément passer tous les VLAN. Toujours vérifier la liste des VLAN autorisés.

### 5. Port G0/0/2 de l'ISR4331 non proposé pour un câble cuivre
- **Cause** : ce port du modèle ISR4331 dans Packet Tracer n'acceptait pas le câble cuivre attendu, et le module SFP essayé était incompatible.
- **Solution** : remplacement de R-SIEGE par un **2911** (3 ports Gigabit cuivre), puis vérification du câblage avec `show cdp neighbors`.
- **Leçon** : `show cdp neighbors` permet de cartographier les liaisons sans voir les câbles.

### 6. Premiers pings perdus
- **Cause** : la résolution ARP sur chaque saut au premier envoi. Comportement normal.

### 7. Module Wi-Fi refusé sur le laptop
- **Cause** : le WPC300N est prévu pour les PC de bureau, et le laptop était allumé.
- **Solution** : laptop éteint, carte Ethernet retirée, module **PT-LAPTOP-NM-1W**.
