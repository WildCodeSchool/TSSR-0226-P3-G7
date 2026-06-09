# configuration.md — Configuration de pfSense FW01

> **Type** : LLD — Low Level Design  
> **Dossier** : `components/fw01/`  
> **Sprint** : Sprint 02  
> **Statut** : Terminé  
> **Dernière mise à jour** : 08/06/2026 
> **Auteur** : Patrick TAMBWE

---

## Sommaire

- [1. Interfaces réseau configurées](#1-interfaces-réseau-configurées)
- [2. Configuration DHCP](#2-configuration-dhcp)
- [3. Configuration DNS](#3-configuration-dns)
- [4. Règles pare-feu](#4-règles-pare-feu)
- [5. Ports utilisés](#5-ports-utilisés)
- [6. Comptes créés](#6-comptes-créés)
- [7. Flux inter-zones](#7-flux-inter-zones)
- [8. Test de connectivité](#8-test-de-connectivité)

---

## 1. Interfaces réseau configurées

| Interface | Nom pfSense | IP | Masque | Rôle |
|-----------|-------------|-----|--------|------|
| em0 | WAN | 10.0.2.15 (DHCP NAT) | /24 | Connexion internet via box FAI |
| em1 | LAN | 172.16.10.254 | /24 | Passerelle réseau interne |
| em2 | OPT1 (DMZ) | 172.16.20.254 | /24 | Passerelle zone exposée |

### Détail WAN
- **Mode** : DHCP Client (reçoit automatiquement une IP du NAT VirtualBox)
- **IP obtenue** : 10.0.2.15/24
- **Passerelle** : 10.0.2.2 (passerelle NAT VirtualBox)
- **VLAN** : Désactivé

### Détail LAN
- **Mode** : Statique
- **IP** : 172.16.10.254/24
- **DHCP** : Activé (voir section 2)
- **Réseau VirtualBox** : `LAN-Pharmgreen`

### Détail DMZ (OPT1)
- **Mode** : Statique
- **IP** : 172.16.20.254/24
- **DHCP** : Désactivé (IPs statiques uniquement en DMZ)
- **Réseau VirtualBox** : `DMZ-Pharmgreen`

---

## 2. Configuration DHCP

Le serveur DHCP est activé uniquement sur l'interface **LAN**.

| Paramètre | Valeur |
|-----------|--------|
| Interface | LAN (em1) |
| Plage dynamique | 172.16.10.10 -> 172.16.10.200 |
| Passerelle distribuée | 172.16.10.254 |
| DNS distribué | 172.16.10.254 |
| Adresses réservées serveurs | 172.16.10.1 -> 172.16.10.9 |
| Adresses libres (extensions) | 172.16.10.201 -> 172.16.10.253 |

> Le DHCP est **désactivé sur OPT1 (DMZ)** - les serveurs en DMZ ont des IPs statiques.

### Bail DHCP obtenu par CLIWIN02

| Paramètre | Valeur |
|-----------|--------|
| IP attribuée | 172.16.10.11 |
| Masque | 255.255.255.0 |
| Passerelle | 172.16.10.254 |
| DNS | 172.16.10.254 |
| Durée du bail | 2 heures |

---

## 3. Configuration DNS

pfSense agit comme **DNS Resolver** pour les clients du LAN.

| Paramètre | Valeur |
|-----------|--------|
| DNS Resolver | Activé (mode par défaut pfSense) |
| DNS Forwarders | 8.8.8.8 (Google) · 1.1.1.1 (Cloudflare) |
| DNS distribué aux clients | 172.16.10.254 |

---

## 4. Règles pare-feu

Les règles sont appliquées dans l'ordre - pfSense s'arrête à la première règle correspondante.

Logique de navigation :
  - Onglet WAN = règles pour le trafic qui arrive de l'internet vers pfSense
  - Onglet LAN = règles pour le trafic qui part du réseau interne
  - Onglet OPT1 = règles pour le trafic de la DMZ



### Onglet WAN - Trafic entrant depuis internet

| # | Action | Protocol | Source | Destination | Ports | Description |
|---|--------|----------|--------|-------------|-------|-------------|
| 1 | Block | - | RFC 1918 | - | - | Block private networks (défaut) |
| 2 | Block | - | Bogon | - | - | Block bogon networks (défaut) | -> Adresses IP non assignées
| 3 | Pass | TCP | Any | OPT1 subnets | 80-443 | R02 - WAN vers DMZ HTTP HTTPS |

-> Bogon networks (adresses ou réseaux IP non assignées)

### Onglet LAN — Trafic depuis le réseau interne

| # | Action | Protocol | Source | Destination | Ports | Description |
|---|--------|----------|--------|-------------|-------|-------------|
| 1 | Pass | - | - | LAN Address | 443·80 | Anti-Lockout Rule (défaut) |
| 2 | Pass | IPv4 * | LAN subnets | - | - | Default allow LAN to any rule |
| 3 | Pass | IPv6 * | LAN subnets | - | - | Default allow LAN IPv6 to any rule |

### Onglet OPT1 — Trafic depuis la DMZ

| # | Action | Protocol | Source | Destination | Ports | Description |
|---|--------|----------|--------|-------------|-------|-------------|
| 1 | Block | IPv4 * | OPT1 subnets | LAN subnets | - | R03 - Bloquer DMZ vers LAN |

### Règle implicite (Deny All)

> pfSense a appliqué automatiquement une règle **Deny All implicite** en dernier - tout trafic ne correspondant à aucune règle est silencieusement bloqué.

---

## 5. Ports utilisés

| Service | Port | Protocole | Interface | Direction |
|---------|------|-----------|-----------|-----------|
| Interface web GUI | 443 | HTTPS | LAN | Entrant |
| Interface web GUI | 80 | HTTP | LAN | Entrant (redirigé vers 443) |
| DNS | 53 | UDP/TCP | LAN · DMZ | Sortant |
| DHCP | 67/68 | UDP | LAN | Entrant/Sortant |
| HTTP | 80 | TCP | WAN -> DMZ | Entrant |
| HTTPS | 443 | TCP | WAN -> DMZ | Entrant |
| ICMP (ping) | - | ICMP | LAN | Bidirectionnel |

---

## 6. Comptes créés

| Compte | Rôle | Mot de passe | Interface |
|--------|------|-------------|-----------|
| `admin` | Administrateur pfSense | `Azerty1*` | Web GUI · Console |

> Il convient de signaler que le mot de passe par défaut `pfsense` a été changé en `Azerty1*` après la première connexion.

---

## 7. Flux inter-zones

### Matrice des flux autorisés/bloqués

| Source | Destination | Statut | Règle |
|--------|-------------|--------|-------|
| LAN | WAN (internet) | Autorisé | Default allow LAN to any |
| LAN | DMZ | Autorisé | Default allow LAN to any |
| LAN | FW01 GUI (443) | Autorisé | Anti-Lockout Rule |
| WAN | DMZ (80/443) | Autorisé | R02 |
| WAN | LAN | Bloqué | Deny All implicite |
| DMZ | LAN | Bloqué | R03 |
| DMZ | WAN | Bloqué | Deny All implicite |

### Schéma des flux


<img width="678" height="748" alt="Schema_Flux_Pfsense" src="https://github.com/user-attachments/assets/7caafc7b-73a8-4de8-87ef-15863ebfad77" />


---

## 8. Test de connectivité 

| Test | Résultat | Observation |
|------|----------|------|
| ping 172.16.10.254 (FW01) |  4/4 · perte 0% | Fonctionne |
| ping 8.8.8.8 (internet) |  4/4 · perte 0% | Fonctionne |
| tracert 8.8.8.8 |  1er saut FW01 · atteint Google | Fonctionne |
| Accès GUI https://172.16.10.254 |  Accessible depuis CLIWIN02 | Fonctionne |

---

*Document maintenu par Patrick TAMBWE . Sprint 02 . Pharmgreen. P3_G7 .tssr.lan*

