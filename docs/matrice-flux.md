# Matrice de flux et preuves – Néolys

Principe : **tout est interdit par défaut**, seuls les flux utiles sont autorisés. Les ACL étendues sont appliquées **en entrée**, au plus près de la source (SVI de SW-CORE au siège, sous-interfaces de R-AGENCE à l'agence).

## Matrice

| # | Source | Destination | Décision | Ports |
|---|---|---|---|---|
| 1 | Wi-Fi invités (80) | Internet | ✅ Autorisé | HTTP 80, HTTPS 443, DNS 53 |
| 2 | Wi-Fi invités (80) | Réseaux internes (RFC 1918) | ❌ Refusé | – |
| 3 | Postes (10, 20, 30, 70) | Serveur (40) | ✅ Autorisé | DNS 53, HTTP/HTTPS, SMB 445, LDAP 389, Kerberos 88, ICMP |
| 4 | Postes | Imprimantes (60) | ✅ Autorisé | 9100, IPP 631, ICMP |
| 5 | Imprimantes (60) | Tout | ❌ Refusé (réponses uniquement) | TCP established, echo-reply |
| 6 | Postes | Autres VLAN de postes | ❌ Refusé | – |
| 7 | Téléphones (50) | Téléphonie | ✅ (à affiner avec le serveur de téléphonie) | SIP 5060, RTP |
| 8 | Caméras (90) | Enregistreur (40) | ✅ Autorisé | RTSP 554 |
| 9 | Caméras (90) | Internet | ❌ Refusé | – |
| 10 | Tout VLAN utilisateur | Administration réseau (99) | ❌ Refusé | – |
| 11 | Postes de l'agence | Serveur du siège (40) | ✅ Autorisé | Mêmes ports que la règle 3 |
| 12 | Postes | Internet | ✅ Autorisé | Tout (via NAT) |

## ACL du siège (SW-CORE)

```
ip access-list extended ACL-POSTES
 permit udp any any eq bootps
 permit udp any host 10.10.40.10 eq domain
 permit tcp any host 10.10.40.10 eq www
 permit tcp any host 10.10.40.10 eq 443
 permit tcp any host 10.10.40.10 eq 445
 permit tcp any host 10.10.40.10 eq 389
 permit tcp any host 10.10.40.10 eq 88
 permit icmp any 10.10.40.0 0.0.0.255
 permit tcp any 10.10.60.0 0.0.0.255 eq 9100
 permit tcp any 10.10.60.0 0.0.0.255 eq 631
 permit icmp any 10.10.60.0 0.0.0.255
 deny   ip any 10.0.0.0 0.255.255.255
 permit ip any any
!
ip access-list extended ACL-INVITES
 permit udp any any eq bootps
 permit udp any host 198.51.100.10 eq domain
 deny   ip any 10.0.0.0 0.255.255.255
 deny   ip any 172.16.0.0 0.15.255.255
 deny   ip any 192.168.0.0 0.0.255.255
 permit tcp any any eq www
 permit tcp any any eq 443
 permit icmp any any
 deny   ip any any
!
ip access-list extended ACL-IMPRIMANTES
 permit udp any any eq bootps
 permit tcp any any established
 permit icmp any any echo-reply
 deny   ip any any
!
ip access-list extended ACL-CAMERAS
 permit udp any any eq bootps
 permit tcp any 10.10.40.0 0.0.0.255 eq 554
 permit icmp any any echo-reply
 deny   ip any any
!
interface vlan 10 / 20 / 30 / 70 → ip access-group ACL-POSTES in
interface vlan 80               → ip access-group ACL-INVITES in
interface vlan 60               → ip access-group ACL-IMPRIMANTES in
interface vlan 90               → ip access-group ACL-CAMERAS in
```

Les ACL de l'agence (`AG-POSTES`, `AG-INVITES`, `AG-IMPRIMANTES`) suivent la même logique, sur les sous-interfaces G0/0/0.20, .30, .60 et .80 de R-AGENCE.

**Points de conception**
- La ligne `permit udp any any eq bootps` est placée en tête, sinon le DHCP est bloqué.
- Les ACL sont sans état : `established` et `echo-reply` laissent les imprimantes et les caméras **répondre** sans pouvoir **initier** de connexion.
- La ligne `deny ip any 10.0.0.0 0.255.255.255` bloque d'un coup tout le réseau interne non explicitement autorisé (autres services, administration, caméras…).

## Preuves

| Règle | Test | Attendu | Résultat | Capture |
|---|---|---|---|---|
| 2 | Invité (agence) → `ping 10.10.40.10` | ❌ | ✅ « Destination host unreachable » renvoyé par 10.20.80.1 | [07](../captures/07-test-invites-vers-serveur-bloque.png) |
| 1 | Invité → `http://198.51.100.10` | ✅ | ✅ Page affichée | [08](../captures/08-test-invites-internet-ok.png) |
| 6 | Administratif → `ping 10.10.30.100` (Études) | ❌ | ✅ Refusé par 10.10.20.1 | [09](../captures/09-test-isolation-compta-etudes.png) |
| 12 | Direction → `ping 198.51.100.10` | ✅ | ✅ 4/4 | [10](../captures/10-test-direction-internet.png) |
| 4 | Études → `ping 10.10.60.100` (imprimante) | ✅ | ✅ Réponse | [11](../captures/11-test-etudes-imprimante.png) |
| 11 | Agence → `http://10.10.40.10` | ✅ | ✅ Page affichée | [12](../captures/12-test-agence-vers-glpi.png) |
