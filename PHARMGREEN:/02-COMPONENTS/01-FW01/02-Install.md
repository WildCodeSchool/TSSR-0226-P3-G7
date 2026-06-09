# installation.md - Installation de pfSense FW01

> **Type** : LLD - Low Level Design  
> **Dossier** : `components/fw01/`  
> **Sprint** : Sprint 02  
> **Statut** : Terminé  
> **Dernière mise à jour** : 09/06/2026  
> **Auteur** : Patrick TAMBWE

---

## Sommaire

- [1. Prérequis](#1-prérequis)
- [2. Création de la VM dans VirtualBox](#2-création-de-la-vm-dans-virtualbox)
- [3. Configuration des interfaces réseau](#3-configuration-des-interfaces-réseau)
- [4. Installation de pfSense](#4-installation-de-pfsense)
- [5. Vérification post-installation](#5-vérification-post-installation)

---

## 1. Prérequis

| Élément | Valeur |
|---------|--------|
| Hyperviseur | VirtualBox (installé sur la machine hôte) |
| ISO pfSense | `pfSense-CE-2.8.1-RELEASE-amd64.iso` |
| Espace disque libre | 20 Go minimum |
| RAM disponible | 1 Go minimum allouable |
| Réseaux internes créés | `LAN-Pharmgreen` · `DMZ-Pharmgreen` |

> L'ISO pfSense est distribuée compressée en `.iso.gz` - la décompresser avec 7-Zip avant utilisation.

---

## 2. Création de la VM dans VirtualBox

### 2.1 Paramètres généraux

| Paramètre | Valeur |
|-----------|--------|
| Nom | `FW01` |
| Type | `BSD` |
| Version | `FreeBSD (64-bit)` |
| RAM | `1024 Mo` |
| vCPU | `1` |

> RRMARQUE: Ne pas utiliser l'installation automatisée (Unattended Install); décocher cette option.

### 2.2 Disque dur virtuel

| Paramètre | Valeur |
|-----------|--------|
| Type | `VDI (VirtualBox Disk Image)` |
| Allocation | `Dynamiquement alloué` |
| Taille | `20 Go` |
| Contrôleur | `SATA` (obligatoire - IDE provoque une erreur d'installation) |

> REMARQUE: Le disque doit être sur un contrôleur **SATA** et non IDE. pfSense ne détecte pas les disques IDE sous VirtualBox.

### 2.3 Paramètres système

| Section | Paramètre | Valeur |
|---------|-----------|--------|
| Système - Processeur | PAE/NX | Activé |
| Système - Ordre de démarrage | 1. Optique · 2. Disque dur | Disquette désactivée |
| Affichage | Mémoire vidéo | 16 Mo |

---

## 3. Configuration des interfaces réseau

Les 3 interfaces doivent être configurées **avant** le premier démarrage.

| Adaptateur VirtualBox | Mode | Nom réseau | Interface pfSense | Zone |
|----------------------|------|------------|-------------------|------|
| Adapter 1 | NAT | - | em0 - WAN | Internet |
| Adapter 2 | Réseau interne | `LAN-Pharmgreen` | em1 - LAN | 172.16.10.0/24 |
| Adapter 3 | Réseau interne | `DMZ-Pharmgreen` | em2 - OPT1/DMZ | 172.16.20.0/24 |

> La case **"Activer l'interface réseau"** doit être cochée sur les 3 adaptateurs.  
> Les noms `LAN-Pharmgreen` et `DMZ-Pharmgreen` sont sensibles à la casse - toutes les autres VMs doivent utiliser exactement les mêmes noms.

---

## 4. Installation de pfSense

### 4.1 Démarrage sur l'ISO

1. Monter l'ISO dans le lecteur optique (Stockage - Contrôleur IDE -> icône disque)
2. Démarrer la VM
3. Menu boot -> option **1 - Boot Multi user** -> appuyer sur Entrée

### 4.2 Assistant d'installation

| Écran | Action |
|-------|--------|
| Active Subscription Validation | Cliquer **Install CE** |
| Installation Options | File System : `ZFS` · Partition Scheme : `GPT` -> Continue |
| ZFS Virtual Device Type | Sélectionner `stripe — No Redundancy` -> OK |
| Disk Selection | Cocher `da0 / ada0` (20 Go) avec Barre espace -> OK |
| Last Chance Confirmation | Sélectionner **Yes** -> Entrée |
| Installation en cours | Attendre 2 à 5 minutes |
| Installation Complete | Sélectionner **Reboot** -> Entrée |

> Avertissement : **Action critique au reboot** - dès que l'écran devient noir, éjecter immédiatement l'ISO dans VirtualBox :  
> `Périphériques → Lecteurs optiques → Retirer le disque du lecteur virtuel`  
> Sans cette action, pfSense redémarre sur l'ISO au lieu du disque installé.

### 4.3 Configuration initiale des interfaces (console)

Au premier démarrage depuis le disque, pfSense affiche le menu de configuration :

**Étape 1 - VLANs**
---
Should VLANs be set up now [y|n]? n
---

**Étape 2 - Assignation des interfaces**


| Menu | Action |
|-------|--------|
| Enter the WAN interface name | Saisir **em0** |
| Enter the LAN interface name | Saisir **em1** |
| Enter the OPT1 interface name | Saisir **em2** |
| Enter the OPT2 interface name | Ne rien saisir (Entrée - vide) |

Do you want to proceed?       : y

---

**Étape 3 - Configuration IP du LAN (option 2 du menu)**

Interface à configurer        : 2 (LAN)
Configure IPv4 via DHCP?      : n
IPv4 address                  : 172.16.10.254
Subnet bit count              : 24
Upstream gateway              : (Entrée - vide)
Configure IPv6?               : n
IPv6 address                  : (Entrée - vide)
Enable DHCP on LAN?           : y
DHCP Range Start              : 172.16.10.10
DHCP Range End                : 172.16.10.200
Revert to HTTP?               : n

---

**Étape 4 - Configuration IP de la DMZ (option 2 du menu)**

Interface à configurer        : 3 (OPT1)
Configure IPv4 via DHCP?      : n
IPv4 address                  : 172.16.20.254
Subnet bit count              : 24
Upstream gateway              : (Entrée - vide)
Configure IPv6?               : n
IPv6 address                  : (Entrée - vide)
Enable DHCP on OPT1?          : n
Revert to HTTP?               : n

---

## 5. Vérification post-installation

### 5.1 Menu console pfSense

Après configuration, le menu principal doit afficher :


*** Welcome to pfSense 2.8.1-RELEASE (amd64) on pfSense ***

WAN  (wan)  → em0 → v4/DHCP4: 10.0.2.15/24
LAN  (lan)  → em1 → v4: 172.16.10.254/24
OPT1 (opt1) → em2 → v4: 172.16.20.254/24


---

### 5.2 Tests de connectivité depuis CLIWIN02

| Test | Commande | Résultat attendu |
|------|----------|-----------------|
| Ping FW01 | `ping 172.16.10.254` | 4/4 réponses · perte 0% |
| Ping internet | `ping 8.8.8.8` | 4/4 réponses · perte 0% |
| Tracert internet | `tracert 8.8.8.8` | 1er saut = 172.16.10.254 |

---

### 5.3 Résultats obtenus 

| Action | Résultat |
|--------|--------|
| ping 172.16.10.254 |Réponse : 4/4 · perte 0% · temps moyen 1ms  |
| ping 8.8.8.8  | Réponse : 4/4 · perte 0% · temps moyen 47ms |
| tracert 8.8.8.8 | Saut 1 : pfSense.home.arpa [172.16.10.254] |
                   
-> Saut 7 : dns.google [8.8.8.8]

---

*Document maintenu par Patrick TAMBWE · Sprint 02 ·P3_G7_Pharmgreen · tssr.lan*

