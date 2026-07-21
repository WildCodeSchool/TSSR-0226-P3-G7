# hardware.md : Inventaire du matériel

> **Type** : LLD  (Low Level Design)  
> **Dossier** : `components/`  
> **Projet** : TSSR Projet 3 - Build Your Infra  
> **Client** : Pharmgreen - Lyon  
> **Domaine AD** : `tssr.lan`  

---

## Sommaire

- [1. Environnement de virtualisation](#1-environnement-de-virtualisation)
- [2. Inventaire des machines virtuelles](#2-inventaire-des-machines-virtuelles)
- [3. Configuration détaillée par VM](#3-configuration-détaillée-par-vm)
- [4. Interfaces réseau par VM](#4-interfaces-réseau-par-vm)
- [5. Ressources totales requises](#5-ressources-totales-requises)
- [6. Ordre de démarrage recommandé](#6-ordre-de-démarrage-recommandé)

---

## 1. Environnement de virtualisation

Toute l'infrastructure Pharmgreen est déployée sur un **hyperviseur de type 2** installé sur une machine personnelle. Pas de matériel physique réseau n'est utilisé dans ce projet.

| Élément | Valeur |
|---------|--------|
| Type d'hyperviseur | Type 2 (hosted) |
| Hyperviseur utilisé | VirtualBox |
| Système hôte | Windows 11 |
| RAM machine hôte recommandée | 16 Go minimum |
| RAM machine hôte minimale | 8 Go (avec optimisations) |
| Stockage libre recommandé | 200 Go minimum |
| Connexion internet | Requise (ISOs · WSUS · forwarders DNS) |

### Configuration réseau virtuel dans l'hyperviseur

| Réseau virtuel | Type | Plage IP | Rôle |
|----------------|------|----------|------|
| `vSwitch-LAN` | Réseau interne | 172.16.10.0/24 | Réseau interne serveurs et clients |
| `vSwitch-DMZ` | Réseau interne | 172.16.20.0/24 | Zone exposée - serveur web |
| `vSwitch-WAN` | NAT ou pont | IP box FAI | Connexion internet |

> Architecture réseau complète :
> [`../architecture/network.md`](../architecture/network.md)

---

## 2. Inventaire des machines virtuelles

### Vue d'ensemble : objectifs principaux (8 VMs)

| Nom VM | Type | OS | RAM | vCPU | Disque | Zone | Priorité |
|--------|------|----|-----|------|--------|------|----------|
| FW01 | Pare-feu | pfSense 2.8 | 1 Go | 1 | 20 Go | WAN/LAN/DMZ | Principale |
| SRVWIN01 | Serveur | Windows Server 2025 GUI | 4 Go | 2 | 60 Go | LAN | Principale |
| SRVWIN04 | Serveur | Windows Server 2025 GUI | 4 Go | 2 | 80 Go | LAN | Principale |
| SRVLX01 | Serveur | Debian 13 CLI | 2 Go | 2 | 60 Go | LAN | Principale |
| IPBX01 | Serveur VoIP | AlmaLinux / FreePBX | 2 Go | 1 | 30 Go | LAN | Principale |
| SRVWEB02 | Serveur web | Debian 13 CLI | 1 Go | 1 | 20 Go | DMZ | Principale |
| CLIWIN01 | Client | Windows 11 | 2 Go | 2 | 60 Go | LAN | Principale |
| CLIWIN02 | Client | Windows 11 | 4 Go | 2 | 60 Go | LAN | Principale |

### VMs supplémentaires : objectifs secondaires (3 VMs)

| Nom VM | Type | OS | RAM | vCPU | Disque | Zone | Priorité |
|--------|------|----|-----|------|--------|------|----------|
| SRVWIN02 | Serveur | Windows Server 2022 Core | 2 Go | 2 | 40 Go | LAN | Secondaire |
| SRVWIN03 | Serveur | Windows Server 2022 Core | 2 Go | 2 | 40 Go | LAN | Secondaire |
| SRVWEB01 | Serveur web | Debian 12 CLI | 1 Go | 1 | 20 Go | LAN | Secondaire |

> En cas de ressources limitées, la priorité serait de déployer d'abord les 8 VMs des objectifs principaux. SRVWIN02, SRVWIN03 et SRVWEB01 seront ajoutés uniquement lors du Sprint 09.

---

## 3. Configuration détaillée par VM

---

### FW01 : Pare-feu pfSense

| Paramètre | Valeur |
|-----------|--------|
| **Nom machine** | FW01 |
| **Rôle** | Pare-feu périmétrique · routage inter-zones · Deny All |
| **OS** | pfSense 2.8 (FreeBSD) |
| **RAM** | 1 Go |
| **vCPU** | 1 |
| **Disque** | 20 Go |
| **Interfaces réseau** | 3 (eth0 WAN · eth1 LAN · eth2 DMZ) |
| **Zone** | WAN / LAN / DMZ |
| **Login console** | `admin` / `pfsense` (à changer dès la 1ère connexion) |
| **Accès web GUI** | https://172.16.10.254 (depuis le LAN) |
| **Sprint de déploiement** | Sprint 02 |

> Procédure complète : [`fw01/installation.md`](fw01/installation.md)

---

### SRVWIN01 : Contrôleur de domaine principal

| Paramètre | Valeur |
|-----------|--------|
| **Nom machine** | SRVWIN01 |
| **Rôle** | AD DS (DC principal) · DNS · DHCP |
| **OS** | Windows Server 2022 Standard (GUI) |
| **RAM** | 4 Go |
| **vCPU** | 2 |
| **Disque système** | 60 Go (C:) |
| **Disque données** | 50 Go (D:) - dossiers partagés (objectif secondaire) |
| **Adresse IP** | 172.16.10.1 (statique) |
| **Masque** | 255.255.255.0 (/24) |
| **Passerelle** | 172.16.10.254 (FW01) |
| **DNS** | 127.0.0.1 (lui-même) |
| **Zone** | LAN |
| **Login** | `Administrator` / `Azerty1*`|
| **Domaine** | `tssr.lan` |
| **Rôles FSMO** | Schema Master · Domain Naming Master · PDC Emulator · RID Master · Infrastructure Master |
| **Sprint de déploiement** | Sprint 03 |

> Procédure complète : [`active-directory/installation.md`](active-directory/installation.md)

---

### SRVWIN02 : Contrôleur de domaine secondaire *(objectif secondaire)*

| Paramètre | Valeur |
|-----------|--------|
| **Nom machine** | SRVWIN02 |
| **Rôle** | AD DS (DC secondaire) · DNS |
| **OS** | Windows Server 2022 Standard (Core - sans GUI) |
| **RAM** | 2 Go |
| **vCPU** | 2 |
| **Disque** | 40 Go (C:) |
| **Adresse IP** | 172.16.10.2 (statique) |
| **Masque** | 255.255.255.0 (/24) |
| **Passerelle** | 172.16.10.254 (FW01) |
| **DNS** | 172.16.10.1 (SRVWIN01) |
| **Zone** | LAN |
| **Login** | `Administrator` / `Azerty1*`|
| **Sprint de déploiement** | Sprint 09 |

> Procédure complète : [`active-directory/installation.md`](active-directory/installation.md)

---

### SRVWIN03 : Contrôleur de domaine tertiaire *(objectif secondaire)*

| Paramètre | Valeur |
|-----------|--------|
| **Nom machine** | SRVWIN03 |
| **Rôle** | AD DS (DC tertiaire) · DNS |
| **OS** | Windows Server 2022 Standard (Core - sans GUI) |
| **RAM** | 2 Go |
| **vCPU** | 2 |
| **Disque** | 40 Go (C:) |
| **Adresse IP** | 172.16.10.3 (statique) |
| **Masque** | 255.255.255.0 (/24) |
| **Passerelle** | 172.16.10.254 (FW01) |
| **DNS** | 172.16.10.1 (SRVWIN01) |
| **Zone** | LAN |
| **Login** | `Administrator` / `Azerty1*`|
| **Sprint de déploiement** | Sprint 09 |

> Procédure complète : [`active-directory/installation.md`](active-directory/installation.md)

---

### SRVWIN04 : Serveur WSUS

| Paramètre | Valeur |
|-----------|--------|
| **Nom machine** | SRVWIN04 |
| **Rôle** | WSUS - mises à jour Windows centralisées |
| **OS** | Windows Server 2022 Standard (GUI) |
| **RAM** | 4 Go |
| **vCPU** | 2 |
| **Disque système** | 80 Go (C:) - espace requis pour le stockage des MAJ |
| **Adresse IP** | 172.16.10.4 (statique) |
| **Masque** | 255.255.255.0 (/24) |
| **Passerelle** | 172.16.10.254 (FW01) |
| **DNS** | 172.16.10.1 (SRVWIN01) |
| **Zone** | LAN |
| **Login** | `Administrator` / `Azerty1*` (à changer) |
| **Port WSUS** | 8530 (HTTP) · 8531 (HTTPS) |
| **Sprint de déploiement** | Sprint 06 |

> Procédure complète : [`wsus/installation.md`](wsus/installation.md)

---

### SRVLX01 - Serveur Linux GLPI + Messagerie

| Paramètre | Valeur |
|-----------|--------|
| **Nom machine** | SRVLX01 |
| **Rôle** | GLPI (gestion de parc + ticketing) · iRedMail (messagerie) |
| **OS** | Debian 12 (Bookworm) - CLI sans interface graphique |
| **RAM** | 2 Go |
| **vCPU** | 2 |
| **Disque** | 60 Go (/) |
| **Adresse IP** | 172.16.10.5 (statique) |
| **Masque** | 255.255.255.0 (/24) |
| **Passerelle** | 172.16.10.254 (FW01) |
| **DNS** | 172.16.10.1 (SRVWIN01) |
| **Zone** | LAN |
| **Login** | `root` / `Azerty1*` (à changer) |
| **Services hébergés** | Apache 2 · MariaDB · PHP 8 · GLPI · Zimbra |
| **Ports utilisés** | 80 (GLPI HTTP) · 443 (HTTPS) · 25/143/993/587 (messagerie) |
| **Sprint de déploiement** | Sprint 05 (GLPI) · Sprint 07 (Zimbra) |

>  Procédures complètes : [`glpi/installation.md`](glpi/installation.md) · [`messagerie/installation.md`](messagerie/installation.md)

---

### IPBX01 : Serveur VoIP FreePBX

| Paramètre | Valeur |
|-----------|--------|
| **Nom machine** | IPBX01 |
| **Rôle** | Téléphonie VoIP - serveur FreePBX |
| **OS** | AlmaLinux 8 (inclus dans l'ISO FreePBX) |
| **RAM** | 2 Go |
| **vCPU** | 1 |
| **Disque** | 30 Go (/) |
| **Adresse IP** | 172.16.10.6 (statique) |
| **Masque** | 255.255.255.0 (/24) |
| **Passerelle** | 172.16.10.254 (FW01) |
| **DNS** | 172.16.10.1 (SRVWIN01) |
| **Zone** | LAN |
| **Login web** | `admin` / `Azerty1*` (à changer) |
| **Ports utilisés** | 5060/5061 (SIP) · 10000-20000 UDP (RTP audio) · 80/443 (GUI) |
| **Sprint de déploiement** | Sprint 07 |

>  Procédure complète : [`voip/installation.md`](voip/installation.md)

---

### SRVWEB01 : Serveur web interne *(objectif secondaire)*

| Paramètre | Valeur |
|-----------|--------|
| **Nom machine** | SRVWEB01 |
| **Rôle** | Serveur web interne accessible sur le LAN uniquement |
| **OS** | Debian 13 (Bookworm) - CLI |
| **RAM** | 1 Go |
| **vCPU** | 1 |
| **Disque** | 20 Go (/) |
| **Adresse IP** | 172.16.10.7 (statique) |
| **Masque** | 255.255.255.0 (/24) |
| **Passerelle** | 172.16.10.254 (FW01) |
| **DNS** | 172.16.10.1 (SRVWIN01) |
| **Zone** | LAN |
| **Login** | `root` / `Azerty1*`|
| **Services hébergés** | Apache 2 ou Nginx |
| **Ports utilisés** | 80 (HTTP) · 443 (HTTPS) |
| **Sprint de déploiement** | Sprint 09 |

---

### SRVWEB02 : Serveur web externe (DMZ)

| Paramètre | Valeur |
|-----------|--------|
| **Nom machine** | SRVWEB02 |
| **Rôle** | Serveur web externe accessible depuis internet via la DMZ |
| **OS** | Debian 12 (Bookworm) - CLI |
| **RAM** | 1 Go |
| **vCPU** | 1 |
| **Disque** | 20 Go (/) |
| **Adresse IP** | 172.16.20.1 (statique) |
| **Masque** | 255.255.255.0 (/24) |
| **Passerelle** | 172.16.20.254 (FW01 eth2) |
| **DNS** | 172.16.10.1 (SRVWIN01 via règle pfSense) |
| **Zone** | DMZ |
| **Login** | `root` / `Azerty1*`|
| **Services hébergés** | Apache 2 ou Nginx |
| **Ports exposés** | 80 (HTTP) · 443 (HTTPS) depuis WAN |
| **Sprint de déploiement** | Sprint 09 |

---

### CLIWIN01 : Poste client Windows 11

| Paramètre | Valeur |
|-----------|--------|
| **Nom machine** | CLIWIN01 |
| **Rôle** | Poste client - tests et validation des services |
| **OS** | Windows 11 Professionnel |
| **RAM** | 2 Go |
| **vCPU** | 2 |
| **Disque** | 60 Go (C:) |
| **Adresse IP** | DHCP (plage 172.16.10.10 → 172.16.10.200) |
| **Zone** | LAN |
| **Domaine** | `tssr.lan` |
| **Login local** | `wilder` / `Azerty1*` |
| **Logiciels installés** | Softphone 3CX · client mail |
| **Sprint de déploiement** | Sprint 08 |

> Procédure complète : [`clients/installation.md`](clients/installation.md)

---

### CLIWIN02 : Poste client Windows 11

| Paramètre | Valeur |
|-----------|--------|
| **Nom machine** | CLIWIN02 |
| **Rôle** | Poste client - tests et validation des services |
| **OS** | Windows 11 Professionnel |
| **RAM** | 4 Go |
| **vCPU** | 2 |
| **Disque** | 60 Go (C:) |
| **Adresse IP** | DHCP (plage 172.16.10.10 -> 172.16.10.200) |
| **Zone** | LAN |
| **Domaine** | `tssr.lan` |
| **Login local** | `wilder` / `Azerty1*` |
| **Logiciels installés** | Softphone 3CX · client mail |
| **Sprint de déploiement** | Sprint 08 |

> Procédure complète : [`clients/installation.md`](clients/installation.md)

---

## 4. Interfaces réseau par VM

| VM | Interface | Réseau virtuel | Adresse IP | Masque | Rôle |
|----|-----------|----------------|------------|--------|------|
| FW01 | eth0 | vSwitch-WAN | DHCP box FAI | /24 | Connexion internet |
| FW01 | eth1 | vSwitch-LAN | 172.16.10.254 | /24 | Passerelle LAN |
| FW01 | eth2 | vSwitch-DMZ | 172.16.20.254 | /24 | Passerelle DMZ |
| SRVWIN01 | eth0 | vSwitch-LAN | 172.16.10.1 | /24 | LAN |
| SRVWIN02 | eth0 | vSwitch-LAN | 172.16.10.2 | /24 | LAN |
| SRVWIN03 | eth0 | vSwitch-LAN | 172.16.10.3 | /24 | LAN |
| SRVWIN04 | eth0 | vSwitch-LAN | 172.16.10.4 | /24 | LAN |
| SRVLX01 | eth0 | vSwitch-LAN | 172.16.10.5 | /24 | LAN |
| IPBX01 | eth0 | vSwitch-LAN | 172.16.10.6 | /24 | LAN |
| SRVWEB01 | eth0 | vSwitch-LAN | 172.16.10.7 | /24 | LAN |
| SRVWEB02 | eth0 | vSwitch-DMZ | 172.16.20.1 | /24 | DMZ |
| CLIWIN01 | eth0 | vSwitch-LAN | DHCP .10→.200 | /24 | LAN |
| CLIWIN02 | eth0 | vSwitch-LAN | DHCP .10→.200 | /24 | LAN |

---

## 5. Ressources totales requises

### Objectifs principaux uniquement (8 VMs)

| Ressource | Total requis | Recommandé machine hôte |
|-----------|-------------|------------------------|
| RAM totale VMs | 20 Go | 24 Go (hôte 8 Go + VMs 16 Go actives max) |
| Stockage total | 410 Go | 450 Go libres |
| vCPU total | 13 | 6 cœurs physiques minimum |

### Avec objectifs secondaires (11 VMs)

| Ressource | Total requis | Recommandé machine hôte |
|-----------|-------------|------------------------|
| RAM totale VMs | 25 Go | 32 Go (hôte + VMs) |
| Stockage total | 510 Go | 550 Go libres |
| vCPU total | 17 | 8 cœurs physiques |

### Stratégie de démarrage partiel (RAM limitée)

| Tâche en cours | VMs à allumer | RAM nécessaire |
|---------------|---------------|----------------|
| Configuration pfSense | FW01 | 1 Go |
| Configuration AD DS | FW01 + SRVWIN01 | 5 Go |
| Tests GPO | FW01 + SRVWIN01 + CLIWIN01 | 7 Go |
| Tests WSUS | FW01 + SRVWIN01 + SRVWIN04 + CLIWIN01 | 11 Go |
| Tests VoIP | FW01 + SRVWIN01 + IPBX01 + CLIWIN01 + CLIWIN02 | 11 Go |
| Tests messagerie | FW01 + SRVWIN01 + SRVLX01 + CLIWIN01 + CLIWIN02 | 11 Go |
| Tests complets | FW01 + SRVWIN01 + SRVLX01 + CLIWIN01 + CLIWIN02 | 11 Go |

---

## 6. Ordre de démarrage recommandé

Le démarrage des VMs doit toujours respecter l'ordre suivant pour éviter les erreurs de résolution DNS et d'authentification AD.

```
1. FW01          → pare-feu et routage (toujours en premier)
2. SRVWIN01      → AD DS + DNS + DHCP (avant tout autre serveur)
3. SRVWIN02/03   → réplication AD (après SRVWIN01 stabilisé)
4. SRVWIN04      → WSUS (après AD DS)
5. SRVLX01       → GLPI + Zimbra (après DNS)
6. IPBX01        → VoIP (après DNS)
7. SRVWEB01/02   → web interne/externe (après DNS)
8. CLIWIN01/02   → clients (en dernier — après tous les services)
```

> Recommandation: Ne jamais démarrer les postes clients avant que FW01 et SRVWIN01 soient complètement démarrés et opérationnels. Sans DNS et DHCP fonctionnels, les clients ne peuvent pas rejoindre le domaine.

---

