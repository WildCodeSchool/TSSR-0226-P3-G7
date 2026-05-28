# overview.md — Vue d'ensemble de l'infrastructure

> **Type** : HLD - High Level Design  
> **Dossier** : `architecture/`  
> **Projet** : TSSR Projet 3 - Build Your Infra  
> **Client** : Pharmgreen - Lyon  
> **Domaine AD** : `tssr.lan`  
> **Dernière mise à jour** : 28/05/2026  
> **Auteur** : Patrick TAMBWE

---

## Sommaire

- [1. Résumé du projet](#1-résumé-du-projet)
- [2. Objectifs globaux](#2-objectifs-globaux)
- [3. Schéma global de l'infrastructure](#3-schéma-global-de-linfrastructure)
- [4. Liste des briques techniques](#4-liste-des-briques-techniques)
- [5. Liens vers les autres fichiers HLD](#5-liens-vers-les-autres-fichiers-hld)

---

## 1. Résumé du projet

Pharmgreen est une entreprise lyonnaise de **211 collaborateurs** répartis dans **11 départements**, spécialisée dans la fabrication de dispositifs médicaux d'origine végétale. Elle ne dispose d'aucune infrastructure IT structurée à ce jour : pas de serveur dédié, pas de domaine, pas de sécurité centralisée, tous les postes fonctionnant en workgroup sur un réseau Wi-Fi fourni par une simple box FAI.

Dans le cadre d'une mission de prestation, notre équipe a été mandatée par le DSI pour **concevoir, déployer et documenter une infrastructure réseau professionnelle complète** répondant aux besoins de centralisation des identités, de sécurisation périmétrique, de gestion des services internes et d'unification de la téléphonie et de la messagerie.

L'infrastructure cible repose sur **une architecture virtualisée en trois zones distinctes** — WAN, LAN et DMZ — pilotée par un pare-feu pfSense central, avec un domaine Active Directory `tssr.lan` comme épine dorsale de l'ensemble des services.

Le projet se déroule sur **10 semaines** organisées en **10 sprints hebdomadaires**, chacun produisant une documentation technique à jour sur ce dépôt GitHub.

---

## 2. Objectifs globaux

### Objectifs principaux - obligatoires à 100 %

| # | Objectif | Brique technique | Statut |
|---|----------|-----------------|--------|
| 01 | Sécuriser le périmètre réseau avec 3 zones WAN / LAN / DMZ | FW01 — pfSense | En cours |
| 02 | Centraliser les identités et authentifications | SRVWIN01 — AD DS | A faire |
| 03 | Assurer la résolution DNS interne et externe | SRVWIN01 — DNS | A faire |
| 04 | Distribuer automatiquement les adresses IP | SRVWIN01 — DHCP | A faire |
| 05 | Gérer le parc informatique et les incidents | SRVLX01 — GLPI | A faire |
| 06 | Centraliser et contrôler les mises à jour | SRVWIN04 — WSUS | A faire |
| 07 | Unifier la téléphonie en VoIP interne | IPBX01 — FreePBX | A faire |
| 08 | Fournir une messagerie interne professionnelle | SRVLX01 — Zimbra | A faire |
| 09 | Intégrer les postes clients au domaine | CLIWIN01 / CLIWIN02 | A faire |

---

### Objectifs secondaires - minimum 50 % requis

| # | Objectif | Brique technique |
|---|----------|-----------------|
| S01 | Dossiers partagés avec mappage lecteur I: | SRVWIN01 — 2e disque |
| S02 | Redondance du contrôleur de domaine | SRVWIN02 / SRVWIN03 — Core |
| S03 | Restriction horaire des connexions (7h30–20h, lun–sam) | GPO AD DS |
| S04 | Synchronisation NTP | SRVWIN01 |
| S05 | Déploiement WSUS via GPO | SRVWIN04 + GPO |
| S06 | Serveur web interne (LAN) | SRVWEB01 — Debian |
| S07 | Serveur web externe (DMZ) | SRVWEB02 — Debian |
| S08 | Synchronisation GLPI ↔ Active Directory | SRVLX01 |
| S09 | Switches virtuels et routeur VyOS | Hyperviseur |

---

## 3. Schéma de l'infrastructure

<img width="667" height="755" alt="SCHEMA_INFRA_VFpng" src="https://github.com/user-attachments/assets/fc7f03fa-8e70-4b83-ba86-ceeb04076f79" />


### Flux inter-zones - règles fondamentales

| Flux | Direction | Autorisé | Remarque |
|------|-----------|----------|----------|
| Clients internes | LAN → WAN | Oui | Accès internet |
| Clients internes | LAN → DMZ | Oui | Accès au site web externe |
| Serveur web externe | DMZ → LAN | Non | Isolation DMZ |
| Internet | WAN → DMZ (80/443) | Oui | Accès site web public |
| Internet | WAN → LAN | Non | Protection réseau interne |
| Administration | LAN → FW01 (443) | Oui | Console web pfSense |


