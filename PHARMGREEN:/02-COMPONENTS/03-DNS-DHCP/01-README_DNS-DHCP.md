# DNS & DHCP

> Brique DNS et DHCP du projet Pharmgreen 
> Serveur : SRVWIN01 · Domaine : tssr.lan · Sprint 03

---

## Introduction

Le DNS et le DHCP sont deux services fondamentaux de toute infrastructure réseau.

Le DNS traduit les noms de machines en adresses IP - sans lui, aucune communication par nom n'est possible dans le domaine.

Le DHCP automatise la distribution des adresses IP aux postes clients - sans lui, chaque machine devrait être configurée manuellement.

Dans cette infrastructure, ces deux rôles sont hébergés sur `SRVWIN01`, le contrôleur de domaine principal, ce qui garantit leur disponibilité dès le démarrage du domaine tssr.lan.

Ensemble, ils forment le socle invisible mais indispensable sur lequel repose toute la communication entre les machines de l'infrastructure Pharmgreen.



## Rôle dans l'infrastructure

Le serveur **SRVWIN01** héberge deux rôles complémentaires :

- **DNS** - résout les noms de machines du domaine `tssr.lan` et transfère les requêtes internet vers les serveurs `Google (8.8.8.8)` et Cloudflare (1.1.1.1)
- **DHCP** - distribue automatiquement les adresses IP aux postes clients du réseau `LAN (172.16.10.10 -> 172.16.10.200)`

Ces deux rôles ont été installés automatiquement avec le rôle **AD DS** lors de la promotion de `SRVWIN01` en contrôleur de domaine.

---

## Informations techniques

| Paramètre | Valeur |
|---|---|
| Serveur | SRVWIN01 |
| Adresse IP | 172.16.10.1 |
| Zone DNS principale | tssr.lan |
| Plage DHCP | 172.16.10.10 -> 172.16.10.200 |
| Passerelle DHCP | 172.16.10.254 (FW01) |
| Forwarders | 8.8.8.8 · 1.1.1.1 |
| Sprint | Sprint 03 |

---

## Contenu du dossier

| Fichier | Description |
|---|---|
| `README.md` | Ce fichier - présentation de la brique DNS-DHCP |
| `configuration.md` | Configuration détaillée - zones, enregistrements A, forwarders, étendue DHCP |

---

## Dépendances

| Dépendance | Rôle |
|---|---|
| FW01 pfSense | Passerelle par défaut distribuée aux clients DHCP |
| SRVWIN01 AD DS | Le DNS est intégré à l'Active Directory (zone AD-Integrated) |

---

## Liens

- [configuration.md](configuration.md)
- [active-directory/configuration.md](../active-directory/configuration.md)
- [fw01/configuration.md](../fw01/configuration.md)

