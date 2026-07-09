# INSTALLATION - FW01 (pfSense CE)

> **Type** : LLD - Low Level Design  
> **Dossier** : `components/fw01/`  
> **Sprint** : Sprint 02  
> **Statut** : Terminé  

---

## Sommaire 

- [1. Prérequis](#1-prérequis)
- [2. Création de la VM dans VirtualBox](#2-création-de-la-vm-dans-virtualbox)
- [3. Installation de pfSense CE](#3-installation-de-pfsense-ce)
- [4. Accès à interface web](#4-acces-à-interface-web)
- [5. Régles de pare-feu créées](#5-regles-de-pare-feu-creees)
- [6. Vérification après installation](#6-verification-apres-installation)

---

## 1 Prérequis

---

| Élément | Valeur |
|---------|--------|
| Hyperviseur | VirtualBox (installé sur la machine hôte) |
| ISO pfSense | `pfSense-CE-2.8.1-RELEASE-amd64.iso` |
| Espace disque libre | 20 Go minimum |
| RAM disponible | 1 Go minimum allouable |
| Réseaux internes créés | `LAN-Pharmgreen` · `DMZ-Pharmgreen` |

> L'ISO pfSense est distribuée compressée en `.iso.gz` - la décompresser avec 7-Zip avant utilisation.

---

--------------------------------------------------------------------------------------------------------




## 2 Création de la VM dans VirtualBox

**Hyperviseur** : VirtualBox 7.2

### 2.1 Paramètres généraux

| Champ | Valeur |
|---|---|
| Nom | FW01 |
| Type | Other |
| Version | Other/Unknown 64-bit *(pfSense = FreeBSD, non listé nativement)* |
| RAM | 1 Go (minimum), confort recommandé : 2 Go |
| CPU | 1-2 vCPU |
| Disque | 20 Go |
| Contrôleur de stockage | **SATA** *(obligatoire - un contrôleur IDE n'est pas détecté correctement par pfSense sous VirtualBox)* |

> Ne pas monter le disque sur un contrôleur IDE - pfSense CE échoue à détecter le disque dans cette configuration sous VirtualBox.

### 2.2 Configuration réseau - 3 interfaces (à faire AVANT le premier démarrage)

| Adaptateur VirtualBox | Mode | Nom réseau | Interface pfSense (attendue) | Zone |
|---|---|---|---|---|
| Adaptateur 1 | NAT | - | em0 -> WAN | Accès Internet |
| Adaptateur 2 | Réseau interne | `LAN-Pharmgreen` | em1 -> LAN | 172.16.10.0/24 |
| Adaptateur 3 | Réseau interne | `DMZ-Pharmgreen` | em2 -> OPT1 (DMZ) | 172.16.20.0/24 |

> Cocher **"Activer l'interface réseau"** sur les 3 adaptateurs.
> Les noms `LAN-Pharmgreen` et `DMZ-Pharmgreen` sont sensibles à la casse ; toutes les autres VM du projet doivent utiliser exactement les mêmes noms de réseau interne pour communiquer.

### 2.3 Paramètres système complémentaires

| Section | Paramètre | Valeur |
|---|---|---|
| Système -> Processeur | PAE/NX | Activé |
| Système -> Ordre de démarrage | 1. Optique · 2. Disque dur | Disquette désactivée |
| Affichage | Mémoire vidéo | 16 Mo |

### 2.4 Monter l'ISO

**Stockage -> Contrôleur IDE** (lecteur optique) -> sélectionner `pfSense-CE-2.8.1-RELEASE-amd64.iso`.

---

## 3. Installation de pfSense CE

### 3.1 Démarrage sur l'ISO

Démarrer la VM. Au menu de boot, sélectionner **1 - Boot Multi user** -> Entrée.

### 3.2 Assistant d'installation

| Écran | Action |
|---|---|
| Active Subscription Validation | **Install CE** *(Community Edition - pas Plus)* |
| Installation Options | File System : **ZFS** · Partition Scheme : **GPT** -> Continue |
| ZFS Virtual Device Type | **stripe - No Redundancy** -> OK |
| Disk Selection | Cocher le disque `da0`/`ada0` (20 Go) avec la barre espace -> OK |
| Last Chance Confirmation | **Yes** -> Entrée |
| Installation en cours | Attendre 2 à 5 minutes |
| Installation Complete | **Reboot** -> Entrée |

> **Action critique au reboot** : dès que l'écran devient noir, éjecter immédiatement l'ISO dans VirtualBox (**Périphériques -> Lecteurs optiques -> Retirer le disque du lecteur virtuel**). Sans cette action, pfSense redémarre en boucle sur l'ISO au lieu du disque installé.

### 3.3 Configuration initiale des interfaces (console)

Au premier démarrage depuis le disque, le menu de configuration console s'affiche :

**Étape 1 : VLANs**
```
Should VLANs be set up now [y|n]? n
```

**Étape 2 : Assignation des interfaces**
```
Enter the WAN interface name  : em0
Enter the LAN interface name  : em1
Enter the OPT1 interface name : em2
Enter the OPT2 interface name : (Entrée - vide)
Do you want to proceed?       : y
```

**Étape 3 : Configuration IP du LAN** (option **2** du menu principal)
```
Interface à configurer   : 2 (LAN)
Configure IPv4 via DHCP? : n
IPv4 address             : 172.16.10.254
Subnet bit count         : 24
Upstream gateway         : (Entrée - vide, pas de passerelle sur le LAN)
Configure IPv6?          : n
Enable DHCP on LAN?      : n  (le DHCP sera pris en charge par SRVWIN01, pas par pfSense)
```

**Étape 4 : Configuration IP du DMZ (OPT1)** (option **2** du menu, choisir l'interface OPT1)
```
IPv4 address : 172.16.20.254
Subnet bit count : 24
Enable DHCP on OPT1? : n
```

L'interface **WAN (em0)** reste en obtention automatique d'adresse (DHCP via le moteur NAT de VirtualBox) - aucune configuration manuelle nécessaire.

---

## 4 Accès à interface web

Depuis un poste connecté au réseau `LAN-Pharmgreen` (ex. CLIWIN02), ouvrir un navigateur vers :
```
https://172.16.10.254
```
*(certificat auto-signé : accepter l'exception de sécurité)*

Identifiants par défaut à la première connexion :
```
Utilisateur : admin
Mot de passe : pfsense
```

L'assistant de configuration initiale (Setup Wizard) propose ensuite de changer ce mot de passe et de définir le nom d'hôte / domaine , étapes validées selon le cahier de charge du projet (nom d'hôte `FW01`, domaine `tssr.lan`).

---

## 5 Règles de pare-feu créées

| Règle | Interface | Source | Destination | Port | Action |
|---|---|---|---|---|---|
| R01 | LAN | LAN net | any | any | Allow *(règle par défaut)* |
| R02 | WAN | any | OPT1 subnets (DMZ) | 80, 443 | Allow |
| R03 | OPT1 (DMZ) | DMZ net | LAN net | any | Block |
| R04 | *(implicite)* | any | any | any | Deny All *(règle implicite de pfSense en fin de liste)* |

> **Ordre de démarrage impératif** : FW01 doit toujours démarrer en premier dans l'infrastructure. Sans lui, aucune VM du LAN n'obtient de bail DHCP (une fois SRVWIN01 configuré comme serveur DHCP) et ne peut router vers l'extérieur.

---

## 6 Vérification après installation

### 6.1 Menu console pfSense

Après configuration, le menu principal doit afficher :


*** Welcome to pfSense 2.8.1-RELEASE (amd64) on pfSense ***

WAN  (wan)  -> em0 -> v4/DHCP4: 10.0.2.15/24
LAN  (lan)  -> em1 -> v4: 172.16.10.254/24
OPT1 (opt1) -> em2 -> v4: 172.16.20.254/24


---

### 6.2 Tests de connectivité depuis CLIWIN02

| Test | Commande | Résultat attendu |
|------|----------|-----------------|
| Ping FW01 | `ping 172.16.10.254` | 4/4 réponses · perte 0% |
| Ping internet | `ping 8.8.8.8` | 4/4 réponses · perte 0% |
| Tracert internet | `tracert 8.8.8.8` | 1er saut = 172.16.10.254 |

---

### 6.3 Résultats obtenus 

| Action | Résultat |
|--------|--------|
| ping 172.16.10.254 |Réponse : 4/4 · perte 0% · temps moyen 1ms  |
| ping 8.8.8.8  | Réponse : 4/4 · perte 0% · temps moyen 47ms |
| tracert 8.8.8.8 | Saut 1 : pfSense.home.arpa [172.16.10.254] |
                   
-> Saut 7 : dns.google [8.8.8.8]

---
