# Plan d'adressage et VLAN – Néolys

## Règle de construction

`10.SITE.VLAN.0/24`
- **SITE** : 10 pour le siège, 20 pour l'agence de Belfort, puis 30, 40… pour les futurs sites.
- **VLAN** : le 3e octet reprend le numéro du VLAN, ce qui permet de lire le site et l'usage directement dans l'adresse.

Dans chaque /24 :

| Plage | Usage |
|---|---|
| .1 | Passerelle (SVI ou sous-interface) |
| .2 – .99 | Adresses fixes (serveurs, équipements réseau, imprimantes réservées) |
| .100 – .200 | DHCP |
| .201 – .254 | Réserve |

**Évolutivité** : un nouveau site devient `10.30.0.0/16`, un nouveau VLAN prend un numéro libre (100, 110…). Aucune renumérotation de l'existant.

## Table des VLAN

| VLAN | Nom | Siège | Agence | Passerelle | DHCP | Usage |
|---|---|---|---|---|---|---|
| 10 | DIRECTION | 10.10.10.0/24 | – | .1 | Oui | Direction |
| 20 | ADMINISTRATIF | 10.10.20.0/24 | 10.20.20.0/24 | .1 | Oui | Comptabilité, RH |
| 30 | ETUDES | 10.10.30.0/24 | 10.20.30.0/24 | .1 | Oui | Production, bureau d'études |
| 40 | SERVEURS | 10.10.40.0/24 | – | .1 | Non (fixe) | GLPI, fichiers, AD, enregistreur vidéo |
| 50 | VOIP | 10.10.50.0/24 | 10.20.50.0/24 | .1 | Oui | Téléphones IP (VLAN voix) |
| 60 | IMPRIMANTES | 10.10.60.0/24 | 10.20.60.0/24 | .1 | Oui | Imprimantes |
| 70 | WIFI-SALARIES | 10.10.70.0/24 | – | .1 | Oui | Portables et téléphones professionnels |
| 80 | WIFI-INVITES | 10.10.80.0/24 | 10.20.80.0/24 | .1 | Oui | Visiteurs, Internet uniquement |
| 90 | CAMERAS | 10.10.90.0/24 | – | .1 | Oui | Vidéosurveillance |
| 99 | ADMIN-RESEAU | 10.10.99.0/24 | 10.20.99.0/24 | .1 | Non (fixe) | Gestion des équipements réseau |
| 999 | NATIF-INUTILISE | – | – | – | – | VLAN natif des trunks (contre le VLAN hopping) |

## Liaisons point à point

| Liaison | Réseau | Côté A | Côté B |
|---|---|---|---|
| SW-CORE ↔ R-SIEGE | 10.255.1.0/30 | SW-CORE G1/0/24 : 10.255.1.1 | R-SIEGE G0/0 : 10.255.1.2 |
| R-SIEGE ↔ R-AGENCE | 10.255.0.0/30 | R-SIEGE G0/1 : 10.255.0.1 | R-AGENCE G0/0/1 : 10.255.0.2 |
| R-SIEGE ↔ FAI | 203.0.113.0/30 | R-SIEGE G0/2 : 203.0.113.1 | R-INTERNET G0/0/0 : 203.0.113.2 |

## Adresses fixes

| Équipement | Adresse |
|---|---|
| SRV-GLPI | 10.10.40.10 |
| PC-ADMIN-RESEAU | 10.10.99.10 |
| SRV-WEB-INTERNET (simulation) | 198.51.100.10 |

## Équipements et ports

### Siège
| Équipement | Port | Vers | Mode | VLAN |
|---|---|---|---|---|
| SW-CORE | G1/0/1 | SW-ACC1 G0/1 | Trunk | natif 999 |
| SW-CORE | G1/0/2 | SW-ACC2 G0/1 | Trunk | natif 999 |
| SW-CORE | G1/0/10 | SRV-GLPI | Accès | 40 |
| SW-CORE | G1/0/24 | R-SIEGE G0/0 | Routé | – |
| SW-ACC1 | Fa0/1 | PC-DIRECTION | Accès | 10 |
| SW-ACC1 | Fa0/2 | PC-ADMINISTRATIF | Accès | 20 |
| SW-ACC1 | Fa0/3 | PC-ETUDES | Accès | 30 |
| SW-ACC1 | Fa0/4 | TEL-SIEGE | Accès / voix | 50 |
| SW-ACC2 | Fa0/1 | IMPRIMANTE-SIEGE | Accès | 60 |
| SW-ACC2 | Fa0/2 | AP-SALARIES | Accès | 70 |
| SW-ACC2 | Fa0/3 | AP-INVITES | Accès | 80 |
| SW-ACC2 | Fa0/4 | CAMERA | Accès | 90 |
| SW-ACC2 | Fa0/24 | PC-ADMIN-RESEAU | Accès | 99 |

### Agence
| Équipement | Port | Vers | Mode | VLAN |
|---|---|---|---|---|
| R-AGENCE | G0/0/0 (sous-interfaces .20 à .99) | SW-AGENCE G0/1 | Trunk | natif 999 |
| SW-AGENCE | Fa0/1 | PC-ADMIN-AG | Accès | 20 |
| SW-AGENCE | Fa0/2 | PC-ETUDES-AG | Accès | 30 |
| SW-AGENCE | Fa0/3 | TEL-AGENCE | Voix | 50 |
| SW-AGENCE | Fa0/4 | IMPRIMANTE-AG | Accès | 60 |
| SW-AGENCE | Fa0/5 | AP-INVITES-AG | Accès | 80 |
