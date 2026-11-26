# Plan de déploiement – Nouveau réseau segmenté de Néolys

## Objectif
Passer d'un réseau « à plat » (tous les équipements dans le même réseau) à un réseau segmenté en VLAN sur deux sites, **sans interruption pendant les heures de travail**.

## Pré-requis (J-15 à J-1)
- Validation de la direction (présentation et budget).
- Commande et réception du matériel : 1 switch de niveau 3, 3 switchs d'accès, 1 routeur pour l'agence, points d'accès Wi-Fi.
- **Préconfiguration** de tous les équipements en atelier, à partir de la maquette validée (fichier Packet Tracer).
- **Sauvegarde** des configurations actuelles (switchs, box, serveurs) et export du DHCP actuel.
- Inventaire des équipements à adresse fixe (imprimantes, serveurs, caméras) : nouvelle adresse prévue pour chacun.
- Communication aux salariés : date, durée de la coupure, personne à contacter.

## Phases

| Phase | Quand | Durée | Coupure | Actions |
|---|---|---|---|---|
| 1. Cœur du réseau | Vendredi 18 h | 1 h | Oui, siège | Installation de SW-CORE et R-SIEGE, liaison Internet, NAT, routes |
| 2. Serveurs | Vendredi 19 h | 1 h | Oui, serveurs | Passage des serveurs dans le VLAN 40 (nouvelles IP fixes), mise à jour du DNS |
| 3. Switchs d'accès | Vendredi 20 h | 1 h 30 | Oui, postes | Remplacement des switchs, brassage des prises selon le tableau des ports |
| 4. Imprimantes, téléphones, caméras | Samedi 9 h | 2 h | Partielle | Passage dans leurs VLAN, réservations DHCP, tests |
| 5. Wi-Fi | Samedi 11 h | 1 h | Wi-Fi | SSID NEOLYS-PRO (VLAN 70) et NEOLYS-INVITES (VLAN 80) |
| 6. ACL | Samedi 13 h | 1 h | Non | Application des ACL, VLAN par VLAN, avec un test après chacun |
| 7. Agence | Samedi suivant | 3 h | Oui, agence | Routeur, switch, liaison avec le siège, mêmes tests |

## Tests de validation (après chaque phase)
- Chaque appareil reçoit une adresse dans **son** VLAN.
- Postes → serveur : web, partage de fichiers, DNS.
- Postes → Internet.
- Invités → Internet uniquement (le ping vers 10.10.40.10 doit échouer).
- Isolation entre services (Administratif ↛ Études).
- Impression depuis chaque service, appel entre deux téléphones.
- `show access-lists` : les compteurs augmentent sur les bonnes lignes.

Lundi 8 h : un technicien est présent sur site pendant 2 heures pour les premiers utilisateurs.

## Retour arrière
**Critère de décision** : si à 23 h, le vendredi, les postes n'ont pas accès au serveur et à Internet, on revient en arrière.

1. Rebrancher l'ancienne box et les anciens switchs (laissés en place, étiquetés, non démontés).
2. Remettre les anciennes adresses fixes des serveurs (notées dans l'inventaire).
3. Vérifier que les postes retrouvent une adresse avec l'ancien DHCP.
4. Analyser le problème sur la maquette avant de reprogrammer l'intervention.

Durée estimée du retour arrière : **45 minutes**. Les anciens équipements restent en place pendant 2 semaines après la bascule.

## Risques

| Risque | Probabilité | Impact | Mesure |
|---|---|---|---|
| Une ACL bloque un service oublié | Moyenne | Moyen | ACL appliquées en dernier, VLAN par VLAN, avec un test après chacune |
| Un équipement à IP fixe oublié | Moyenne | Faible | Inventaire préalable, scan du réseau avant la bascule |
| Panne d'un équipement neuf | Faible | Élevé | Équipements testés en atelier, retour arrière prêt |
| Une imprimante n'est pas joignable | Moyenne | Faible | Réservation DHCP, test d'impression dans chaque service |
