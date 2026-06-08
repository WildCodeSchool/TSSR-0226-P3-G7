# PHARMGREEN — INFRASTRUCTURE RESEAU 

> **Type** : DAT - Documentation d'Architecture Technique  
> **Formation** : TSSR - Technicien Supérieur Systèmes et Réseaux  
> **Projet** : Projet 3 - Build Your Infra
> **Prestataire** : Patrick TAMBWE  
> **Client** : Pharmgreen - Lyon  
> **Domaine AD** : `tssr.lan`  
> **Durée** : 10 semaines · 10 sprints  
> **Statut** : En cours

---

## Sommaire général

- [1. Présentation du projet](#1-présentation-du-projet)
- [2. Contexte et situation initiale](#2-contexte-et-situation-initiale)
- [3. Objectifs du projet](#3-objectifs-du-projet)
  - [3.1 Objectifs principaux](#31-objectifs-principaux)
  - [3.2 Objectifs secondaires](#32-objectifs-secondaires)
- [4. Infrastructure déployée](#4-infrastructure-déployée)
  - [4.1 Machines virtuelles](#41-machines-virtuelles)
  - [4.2 Architecture réseau](#42-architecture-réseau)
  - [4.3 Plan d'adressage IP](#43-plan-dadressage-ip)
- [5. Services déployés](#5-services-déployés)
- [6. Nomenclature](#6-nomenclature)
- [7. Structure du dépôt](#7-structure-du-dépôt)
  - [7.1 Architecture - HLD](#71-architecture-hld)
  - [7.2 Composants - LLD](#72-composants-lld)
  - [7.3 Exploitation - DEX](#73-exploitation-dex)
  - [7.4 Sprints - Suivi](#74-sprints-suivi)
- [8. Avancement par sprint](#8-avancement-par-sprint)
- [9. Équipe projet](#9-équipe-projet)
- [10. Ressources et liens utiles](#10-ressources-et-liens-utiles)
- [11. Difficultés rencontrées](#11-difficultés-rencontrées)


 ## 1. Présentation du projet

**Pharmgreen** est une entreprise lyonnaise pionnière dans le domaine de la santé, spécialisée dans la conception et la fabrication de dispositifs médicaux d'origine végétale. Elle compte actuellement de **211 collaborateurs** répartis en **11 départements**, tous basés sur le site de Lyon.

Dans le cadre d'une mission de prestation, notre équipe a été mandatée par le DSI de Pharmgreen pour **concevoir, déployer et documenter une infrastructure réseau professionnelle complète**, en remplacement de l'environnement actuel qui ne répond plus aux exigences de sécurité, de fiabilité et de gestion d'une entreprise de cette taille.

Ce dépôt GitHub constitue le **livrable documentaire officiel** de ce projet. Il rassemble l'ensemble de la documentation technique produite au fil des sprints, organisée selon une vision par cycle de vie : conception (HLD), réalisation (LLD), exploitation (DEX) et suivi (Sprints).

| Type | Dossier | Description |
|------|---------|-------------|
| DAT | `README.md` + `naming.md` | Documentation d'architecture technique |
| HLD | `architecture/` | Vue macro - conception globale |
| LLD | `components/` | Vue micro - détail technique par brique |
| DEX | `operations/` | Documentation d'exploitation quotidienne |
| Suivi | `sprints/` | Traçabilité chronologique sprint par sprint |

---

## 2. Contexte et situation initiale

- Avant ce projet, Pharmgreen ne disposait d'aucune infrastructure IT structurée. Le constat dressé en amont du projet révèle plusieurs lacunes critiques.

- Sur le plan réseau, l'ensemble des utilisateurs accède à internet via une simple box FAI et des répéteurs Wi-Fi, sur un réseau unique en `172.16.30.0/24`, sans aucune segmentation ni contrôle des flux. Il n'existe ni pare-feu dédié, ni routeur d'entreprise, ni switch administrable.

- Sur le plan des postes de travail, les 211 collaborateurs utilisent des PC portables de marques très hétérogènes, tous configurés en **groupe de travail (workgroup)** avec des comptes locaux. Le turnover important des stagiaires, alternants et CDD a conduit à une réutilisation des mots de passe, exposant l'entreprise à des risques de sécurité majeurs.

- Sur le plan des services, la messagerie est hébergée en cloud (`prenom.nom@pharmgreen.fr`), les données sont stockées sur un NAS grand public sans redondance pour les encadrants, et sur des services cloud personnels pour les autres collaborateurs. Il n'existe aucun système de gestion de parc, aucune téléphonie unifiée, et aucune politique de sauvegarde formalisée.

- Sur le plan de la sécurité, il n'existe aucun contrôle d'identité centralisé, aucune politique de mots de passe, aucune journalisation des accès, et aucune séparation entre les zones réseau.


| Domaine | Situation initiale |
|---------|--------------------|
| Réseau | Box FAI + répéteurs Wi-Fi · réseau unique 172.16.30.0/24 |
| Postes | 211 PC portables hétérogènes en workgroup local |
| Identité | Comptes locaux · mots de passe partagés et réutilisés |
| Données | NAS grand public sans redondance · drives cloud personnels |
| Sécurité | Aucun pare-feu · aucune politique de mots de passe · aucune journalisation |
| Messagerie | Cloud externe · format `prenom.nom@pharmgreen.fr` |
| Téléphonie | Téléphonie fixe/mobile aléatoire sans unification |
| Sauvegarde | Ponctuelle sur NAS · pas de rétention définie |


---

## 3. Objectifs du projet

### 3.1 Objectifs principaux

| # | Service | Machine | Statut |
|---|---------|---------|--------|
| 01 | Pare-feu périmétrique WAN/LAN/DMZ | FW01 - pfSense | En cours |
| 02 | Contrôleur de domaine Active Directory | SRVWIN01 - Win Server 2022 | A faire |
| 03 | Service DNS interne | SRVWIN01 - Rôle DNS | A faire |
| 04 | Service DHCP pour le LAN | SRVWIN01 - Rôle DHCP | A faire |
| 05 | Gestion de parc et ticketing | SRVLX01 - GLPI | A faire  |
| 06 | Serveur de mises à jour | SRVWIN04 - WSUS | A faire |
| 07 | Téléphonie VoIP | IPBX01 - FreePBX | A faire |
| 08 | Serveur de messagerie interne | SRVLX01 - Zimbra | A faire |
| 09 | Postes clients intégrés au domaine | CLIWIN01 / CLIWIN02 | A faire |


### 3.2 Objectifs secondaires

| # | Objectif | Statut |
|---|----------|--------|
| S01 | Dossiers partagés + mappage lecteur I: | A faire |
| S02 | Redondance AD DS (SRVWIN02 + SRVWIN03 Core) | A faire  |
| S03 | Restriction horaire connexions (7h30–20h lun–sam) | A faire |
| S04 | Synchronisation NTP | A faire |
| S05 | WSUS via GPO | A faire |
| S06 | Serveur web interne LAN | A faire |
| S07 | Serveur web externe DMZ | A faire |
| S08 | Synchronisation GLPI ↔ AD DS | A faire |
| S09 | Switches virtuels + routeur VyOS | A faire |

---


## 4. Infrastructure  déployée

L'infrastructure cible reposera sur **9 machines virtuelles minimum**, hébergées sur un hyperviseur de type 2 (VirtualBox). Chaque VM remplit un ou plusieurs rôles précis, listés ci-dessous sans détail de configuration (voir le dossier [`components/`](./components/) pour le détail technique).


### 4.1 Machines virtuelles

| Nom | OS | Rôle | IP | Zone |
|-----|----|----|-----|------|
| FW01 | pfSense | Pare-feu · routage inter-zones | eth1: 172.16.10.254 · eth2: 172.16.20.254 | WAN/LAN/DMZ |
| SRVWIN01 | Windows Server 2022 GUI | AD DS · DNS · DHCP | 172.16.10.1 | LAN |
| SRVWIN02 | Windows Server 2022 Core | AD DS · DNS (réplique) | 172.16.10.2 | LAN |
| SRVWIN03 | Windows Server 2022 Core | AD DS · DNS (réplique) | 172.16.10.3 | LAN |
| SRVWIN04 | Windows Server 2022 GUI | WSUS | 172.16.10.4 | LAN |
| SRVLX01 | Debian 13 CLI | GLPI · Messagerie Zimbra | 172.16.10.5 | LAN |
| IPBX01 | AlmaLinux / FreePBX | Téléphonie VoIP | 172.16.10.6 | LAN |
| SRVWEB01 | Debian 13 CLI | Serveur web interne | 172.16.10.7 | LAN |
| SRVWEB02 | Debian 13 CLI | Serveur web externe | 172.16.20.1 | DMZ |
| CLIWIN01 | Windows 10 | Poste client | DHCP .10 → .200 | LAN |
| CLIWIN02 | Windows 11 | Poste client | DHCP .10 → .200 | LAN |

---

### 4.2 Architecture réseau

<img width="667" height="755" alt="SCHEMA_INFRA_VFpng" src="https://github.com/user-attachments/assets/f26a3047-6f61-4ac3-a467-fa366f3b3027" />


---

### 4.3. Plan d'adressage IP


| Zone | Réseau | Masque | Passerelle |
|------|--------|--------|------------|
| LAN | 172.16.10.0 | /24 | 172.16.10.254 (FW01 eth1) |
| DMZ | 172.16.20.0 | /24 | 172.16.20.254 (FW01 eth2) |
| WAN | IP box FAI | /24 | IP box FAI |

| Machine | Adresse IP | Zone |
|---------|-----------|------|
| FW01 (eth1 LAN) | 172.16.10.254 | LAN |
| FW01 (eth2 DMZ) | 172.16.20.254 | DMZ |
| SRVWIN01 | 172.16.10.1 | LAN |
| SRVWIN02 | 172.16.10.2 | LAN |
| SRVWIN03 | 172.16.10.3 | LAN |
| SRVWIN04 | 172.16.10.4 | LAN |
| SRVLX01 | 172.16.10.5 | LAN |
| IPBX01 | 172.16.10.6 | LAN |
| SRVWEB01 | 172.16.10.7 | LAN |
| SRVWEB02 | 172.16.20.1 | DMZ |
| CLIWIN01 / CLIWIN02 | DHCP 172.16.10.10 → .200 | LAN |

Pour le plan d'adressage détaillé avec les VLANs, voir [`architecture/ip_configuration.md`](./architecture/ip_configuration.md).


---

## 5. Services déployés


L'infrastructure fournit les services suivants aux utilisateurs et administrateurs de Pharmgreen. Cette liste donne une vision fonctionnelle. En ce qui concerne l'ordre logique de déploiement et les interdépendances, se référer à : [`architecture/services.md`](./architecture/services.md).

  - **Identité et accès** : Le domaine Active Directory `tssr.lan` centralise l'authentification de l'ensemble des 211 utilisateurs. Les comptes, groupes et       politiques de sécurité (GPO) sont gérés depuis SRVWIN01 et répliqués sur SRVWIN02 et SRVWIN03.

  - **Réseau** : Le service DHCP distribue automatiquement les adresses IP aux postes clients dans la plage `172.16.10.10` à `172.16.10.200`. Le service DNS       assure la résolution des noms internes (`tssr.lan`) et externe (via les forwarders vers les DNS publics).

  - **Sécurité périmétrique** : Le pare-feu pfSense (FW01) contrôle l'ensemble des flux entre les trois zones réseau (WAN, LAN, DMZ) selon le principe du       *Deny All* par défaut.

  - **Gestion de parc** : GLPI, accessible depuis n'importe quel poste client via navigateur web, permet la gestion de l'inventaire matériel et logiciel ainsi     que la gestion des tickets d'incident.

  - **Mises à jour** : WSUS centralise et contrôle la distribution des mises à jour de sécurité Windows sur l'ensemble des machines du domaine, organisées en      groupes de déploiement.

  - **Téléphonie** : FreePBX fournit une solution de téléphonie VoIP unifiée avec des numéros internes attribués aux utilisateurs, accessibles depuis un           softphone (3CX) installé sur les postes clients.

  - **Messagerie** : Un serveur Zimbra (ou iRedMail) héberge les boîtes mail internes des collaborateurs au format `prenom.nom@pharmgreen.fr`, accessibles         depuis un client de messagerie ou un webmail.

  - **Web externe** : Un serveur web Apache/Nginx en DMZ expose un site accessible depuis internet, sans accès possible au réseau interne.

---


## 6. Nomenclature

La nomenclature complète est définie dans le fichier [`naming.md`](./naming.md) à la racine du dépôt. En voici les grandes lignes:

  - **Domaine Active Directory** : `tssr.lan`

  - **Machines** : Les serveurs Windows suivent le format `SRVWINxx`, les serveurs Linux `SRVLXxx`, le pare-feu `FWxx`, les postes VoIP `IPBXxx`, et les           clients `CLIWINxx`. La numérotation commence à `01`.

  - **Utilisateurs AD** : Le format retenu est `prenom.nom` en minuscules, sans accent (ex. : `patrick.tambwe`). En cas d'homonyme, un numéro est ajouté en     suffixe (`prenom.nom2`).

  - **Groupes AD** : La logique AGDLP est appliquée. Les groupes globaux suivent le format `GG-Departement` et les groupes de domaine local `GDL-Service-Droit`.

  - **GPO** : Le format est `GPO-[Cible]-[Fonction]-[id]` (ex. : `GPO-DOM-PasswordPolicy-01`).

  - **Messagerie** : `prenom.nom@pharmgreen.fr` (identique au format existant en cloud).


---

## 7. Structure du dépôt 

---

La structure du dépôt est constituée de 4 dossiers principaux :  `Architecture`, `Components` , `Operations` et `Sprints`. Les 4 dossiers sont tous à la racine, au même niveau; aucun ne dépend d'un autre.

<img width="873" height="748" alt="Arborescence_Globale_Dépôt _Github_VF" src="https://github.com/user-attachments/assets/e572c6db-1719-41a0-8791-eb9a17017d55" />

---

### 7.1. Architecture - HLD

Le dossier `architecture/` constitue la vue macroscopique du projet. Il regroupe le contexte client, les choix de conception retenus, le périmètre d'intervention, l'inventaire des machines virtuelles et le schéma réseau zonal WAN/LAN/DMZ.


<img width="806" height="597" alt="Dossier_Architecture" src="https://github.com/user-attachments/assets/23c339ca-cdd3-4ee0-a7e8-0cab2d2008bd" />




