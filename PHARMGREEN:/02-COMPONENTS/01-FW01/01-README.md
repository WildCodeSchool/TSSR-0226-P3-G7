# fw01/ - Pare-feu pfSense

> **Type** : LLD — Low Level Design  
> **Dossier** : `components/fw01/`  
> **Sprint** : Sprint 02  
> **Statut** : Terminé  
> **Dernière mise à jour** : 09/06/2026 
> **Auteur** : Patrick TAMBWE

---

## Présentation

FW01 est le **pare-feu périmétrique** de l'infrastructure Pharmgreen. Il est basé sur **pfSense CE 2.8.1** et assure le filtrage et le routage entre les trois zones réseau : WAN, LAN et DMZ.

C'est la **première brique déployée** du projet car toutes les autres VMs dépendent du réseau qu'il fournit.

---

## Rôle dans l'infrastructure

| Fonction | Détail |
|----------|--------|
| Pare-feu | Filtrage des flux entre WAN · LAN · DMZ |
| Routeur | Routage inter-zones |
| Serveur DHCP | Attribution IP automatique sur le LAN (.10 → .200) |
| Serveur DNS | Résolution DNS pour les clients LAN |
| Principe de sécurité | **Deny All** par défaut - seuls les flux explicitement autorisés passent |

---

## Informations techniques

| Paramètre | Valeur |
|-----------|--------|
| Nom machine | `FW01` |
| OS | pfSense CE 2.8.1-RELEASE (FreeBSD 15.0) |
| RAM | 1 Go |
| vCPU | 1 |
| Disque | 20 Go (VDI · ZFS · GPT) |
| Hyperviseur | VirtualBox |
| Netgate Device ID | b7ba9141c9d402c5483b |

---

## Interfaces réseau

| Interface | Nom pfSense | Réseau VirtualBox | IP | Rôle |
|-----------|-------------|-------------------|----|------|
| Adapter 1 | WAN (em0) | NAT | DHCP · 10.0.2.15/24 | Connexion internet |
| Adapter 2 | LAN (em1) | LAN-Pharmgreen | 172.16.10.254/24 | Réseau interne |
| Adapter 3 | OPT1/DMZ (em2) | DMZ-Pharmgreen | 172.16.20.254/24 | Zone exposée |

---

## Accès à l'interface web

| Paramètre | Valeur |
|-----------|--------|
| URL | `https://172.16.10.254` |
| Login | `admin` |
| Mot de passe | `Azerty1*` |
| Protocole | HTTPS |

> Accessible uniquement depuis le réseau LAN-Pharmgreen (CLIWIN01 ou CLIWIN02).

---

## Liens vers la documentation détaillée

| Fichier | Contenu |
|---------|---------|
| [`installation.md`](installation.md) | Procédure complète de création de la VM et installation de pfSense |
| [`configuration.md`](configuration.md) | Configuration des interfaces, DHCP, DNS et règles pare-feu |

---

## Liens vers l'architecture

| Document | Lien |
|----------|------|
| Plan d'adressage IP | [`../../architecture/ip_configuration.md`](../../architecture/ip_configuration.md) |
| Architecture réseau | [`../../architecture/network.md`](../../architecture/network.md) |
| Politique de sécurité | [`../../architecture/security.md`](../../architecture/security.md) |

---

*Document maintenu par Patrick TAMBWE · Sprint 02 · P3_G7_Pharmgreen · tssr.lan*

