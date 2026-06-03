# planning.md - Planning général du projet

> **Dossier** : `sprints/`  
> **Projet** : TSSR Projet 3 - Build Your Infra  
> **Client** : Pharmgreen - Lyon  
> **Domaine AD** : `tssr.lan`  
> **Durée totale** : 10 semaines · 10 sprints  
> **Dernière mise à jour** : 03/05/2026  
> **Auteur** : Patrick TAMBWE

---

## Sommaire

- [1. Vue d'ensemble des 10 sprints](#1-vue-densemble-des-10-sprints)
- [2. Détail de chaque sprint](#2-détail-de-chaque-sprint)
- [3. Diagramme de Gantt](#3-diagramme-de-gantt)
- [4. Dépendances entre sprints](#4-dépendances-entre-sprints)
- [5. Charge de travail estimée](#5-charge-de-travail-estimée)
- [6. Suivi d'avancement global](#6-suivi-davancement-global)

---

## 1. Vue d'ensemble des 10 sprints

| Sprint | Semaine | Thème principal | Heures est. | Statut |
|--------|---------|-----------------|-------------|--------|
| [Sprint 01](sprint-01/README.md) | S01 | Analyse, planification & documentation | 11h | En cours |
| [Sprint 02](sprint-02/README.md) | S02 | Infrastructure réseau — FW01 pfSense | 10h | En cours |
| [Sprint 03](sprint-03/README.md) | S03 | Domaine AD DS + DNS + DHCP | 26h | A faire |
| [Sprint 04](sprint-04/README.md) | S04 | Stratégies GPO & sécurité | 10h | A faire |
| [Sprint 05](sprint-05/README.md) | S05 | GLPI — gestion de parc | 9h | A faire  |
| [Sprint 06](sprint-06/README.md) | S06 | WSUS — mises à jour | 9h | A faire  |
| [Sprint 07](sprint-07/README.md) | S07 | VoIP FreePBX & messagerie Zimbra | 22h | A faire |
| [Sprint 08](sprint-08/README.md) | S08 | Intégration clients & tests | 10h | A faire  |
| [Sprint 09](sprint-09/README.md) | S09 | Objectifs secondaires & redondance AD | 18h | A faire  |
| [Sprint 10](sprint-10/README.md) | S10 | Finalisation, documentation & présentation | 18h | A faire |
| **TOTAL** | **10 sem.** | | **~143h** | |


---

## 2. Détail de chaque sprint

---

### Sprint 01 - Analyse, planification & documentation
**Semaine 01 · Durée estimée : 11h**

| Élément | Détail |
|---------|--------|
| **Objectif** | Préparer l'ensemble du projet avant tout déploiement |
| **Prérequis** | Aucun |
| **Livrable principal** | Arborescence GitHub · README.md · naming.md · schéma réseau · planning.md |

| # | Tâche | Durée |
|---|-------|-------|
| T01.1 | Analyser le fichier RH - doublons, prestataires Kamera | 2h |
| T01.2 | Définir le plan d'adressage IP complet | 1h |
| T01.3 | Créer le schéma réseau (SVG/PNG) | 2h |
| T01.4 | Rédiger la nomenclature `naming.md` | 1h |
| T01.5 | Créer l'arborescence GitHub complète | 1h |
| T01.6 | Rédiger `README.md` (DAT) et `overview.md` (HLD) | 2h |
| T01.7 | Rédiger `context.md` et `scope.md` | 1h |
| T01.8 | Rédiger `hardware.md` et `software.md` | 1h |

---

### Sprint 02 - Infrastructure réseau - FW01 pfSense
**Semaine 02 · Durée estimée : 10h**

| Élément | Détail |
|---------|--------|
| **Objectif** | Déployer le pare-feu FW01 et sécuriser les trois zones réseau |
| **Prérequis** | Sprint 01 terminé · plan d'adressage IP validé |
| **Livrable principal** | FW01 fonctionnel · règles WAN/LAN/DMZ · documentation `fw01/` |

| # | Tâche | Durée |
|---|-------|-------|
| T02.1 | Créer la VM FW01 (1 Go RAM · 20 Go disque) | 0.5h |
| T02.2 | Installer pfSense depuis ISO | 1h |
| T02.3 | Configurer eth0 WAN (DHCP box FAI) | 0.5h |
| T02.4 | Configurer eth1 LAN (172.16.10.254/24) | 0.5h |
| T02.5 | Configurer eth2 DMZ (172.16.20.254/24) | 0.5h |
| T02.6 | Créer règles pare-feu LAN → WAN | 1h |
| T02.7 | Créer règles WAN → DMZ (ports 80/443) | 1h |
| T02.8 | Appliquer principe Deny All par défaut | 0.5h |
| T02.9 | Tester connectivité inter-zones (ping/tracert) | 1h |
| T02.10 | Documenter `fw01/` (installation + configuration) | 2h |

---

### Sprint 03 - Domaine AD DS + DNS + DHCP
**Semaine 03 · Durée estimée : 26h**

| Élément | Détail |
|---------|--------|
| **Objectif** | Déployer le domaine Active Directory avec DNS, DHCP et import des 208 utilisateurs |
| **Prérequis** | Sprint 02 terminé · FW01 opérationnel · réseau LAN accessible |
| **Livrable principal** | Domaine `tssr.lan` · 208 utilisateurs importés · DNS + DHCP actifs |

| # | Tâche | Durée |
|---|-------|-------|
| T03.1 | Créer VM SRVWIN01 (4 Go RAM · 60 Go disque) | 0.5h |
| T03.2 | Installer Windows Server 2022 GUI | 1h |
| T03.3 | Configurer IP statique 172.16.10.1 et renommer SRVWIN01 | 0.5h |
| T03.4 | Installer et configurer rôle AD DS | 1h |
| T03.5 | Promouvoir DC — créer domaine `tssr.lan` | 1h |
| T03.6 | Créer les 11 OU de département | 2h |
| T03.7 | Créer les sous-OU de service (30+ services) | 2h |
| T03.8 | Créer groupes GG et GDL (logique AGDLP) | 2h |
| T03.9 | Nettoyer fichier RH (exclure Kamera + doublons) | 1h |
| T03.10 | Écrire script PowerShell import 208 utilisateurs | 2h |
| T03.11 | Exécuter et valider l'import AD | 1h |
| T03.12 | Installer rôle DNS + créer zone directe `tssr.lan` | 1h |
| T03.13 | Créer enregistrements A + forwarders (8.8.8.8 / 1.1.1.1) | 1h |
| T03.14 | Installer rôle DHCP + créer étendue LAN .10 → .200 | 1h |
| T03.15 | Configurer exclusions IP (.1 → .9) et réservations | 0.5h |
| T03.16 | Tester attribution DHCP sur CLIWIN01 | 0.5h |
| T03.17 | Documenter `active-directory/` et `dns-dhcp/` | 3h |

---

### Sprint 04 - Stratégies GPO & sécurité
**Semaine 04 · Durée estimée : 10h**

| Élément | Détail |
|---------|--------|
| **Objectif** | Créer et appliquer les 7 GPO obligatoires + 2 personnalisées |
| **Prérequis** | Sprint 03 terminé · domaine `tssr.lan` opérationnel · utilisateurs importés |
| **Livrable principal** | 9 GPO déployées · testées sur CLIWIN01 · documentation `gpo/` |

| # | Tâche | Durée |
|---|-------|-------|
| T04.1 | `GPO-DOM-PasswordPolicy-01` (complexité · longueur min. 8) | 1h |
| T04.2 | `GPO-DOM-AccountLockout-01` (verrouillage après 5 erreurs) | 0.5h |
| T04.3 | `GPO-DOM-ControlPanel-Block-01` (blocage panneau config.) | 0.5h |
| T04.4 | `GPO-DOM-LocalAdmin-01` (compte domaine = admin local) | 1h |
| T04.5 | `GPO-DOM-PowerShell-Security-01` | 1h |
| T04.6 | `GPO-DOM-Custom-01` (au choix) | 1h |
| T04.7 | `GPO-DOM-Custom-02` (au choix) | 1h |
| T04.8 | Tester toutes les GPO sur CLIWIN01 | 2h |
| T04.9 | Documenter `gpo/` | 1h |

---

### Sprint 05 - GLPI - gestion de parc
**Semaine 05 · Durée estimée : 9h**

| Élément | Détail |
|---------|--------|
| **Objectif** | Déployer GLPI sur Debian et configurer la gestion de parc et le ticketing |
| **Prérequis** | Sprint 03 terminé · DNS fonctionnel · réseau LAN accessible |
| **Livrable principal** | GLPI accessible via navigateur depuis CLIWIN01 · parc configuré · ticketing actif |

| # | Tâche | Durée |
|---|-------|-------|
| T05.1 | Créer VM SRVLX01 Debian 12 CLI (2 Go RAM · 60 Go disque) | 1h |
| T05.2 | Configurer IP statique 172.16.10.5 | 0.5h |
| T05.3 | Installer Apache + MariaDB + PHP (LAMP) | 1h |
| T05.4 | Installer et configurer GLPI | 2h |
| T05.5 | Configurer gestion de parc (inventaire matériel) | 1h |
| T05.6 | Configurer système de ticketing (catégories) | 1h |
| T05.7 | Tester accès web GUI depuis CLIWIN01 (port 80) | 0.5h |
| T05.8 | Documenter `glpi/` | 1h |

---

### Sprint 06 - WSUS - mises à jour
**Semaine 06 · Durée estimée : 9h**

| Élément | Détail |
|---------|--------|
| **Objectif** | Déployer WSUS et centraliser les mises à jour de sécurité Windows |
| **Prérequis** | Sprint 03 terminé · AD DS opérationnel · accès internet disponible |
| **Livrable principal** | WSUS opérationnel · groupes de machines configurés · MAJ de sécurité approuvées |

| # | Tâche | Durée |
|---|-------|-------|
| T06.1 | Créer VM SRVWIN04 (4 Go RAM · 80 Go disque) | 0.5h |
| T06.2 | Installer Windows Server 2022 GUI | 1h |
| T06.3 | Configurer IP statique 172.16.10.4 | 0.5h |
| T06.4 | Installer et configurer rôle WSUS | 1h |
| T06.5 | Créer groupes de machines WSUS | 1h |
| T06.6 | Configurer approbation des MAJ de sécurité | 1h |
| T06.7 | Synchroniser WSUS avec Microsoft Update | 1h |
| T06.8 | Tester déploiement MAJ sur CLIWIN01 | 1h |
| T06.9 | Documenter `wsus/` | 1h |

---

### Sprint 07 - VoIP FreePBX & messagerie Zimbra
**Semaine 07 · Durée estimée : 22h**

| Élément | Détail |
|---------|--------|
| **Objectif** | Déployer la téléphonie VoIP FreePBX et la messagerie interne Zimbra |
| **Prérequis** | Sprint 03 terminé · DNS fonctionnel · réseau LAN accessible |
| **Livrable principal** | Appel VoIP testé entre CLIWIN01 et CLIWIN02 · messagerie fonctionnelle avec envoi/réception validés |

**VoIP — FreePBX**

| # | Tâche | Durée |
|---|-------|-------|
| T07.1 | Créer VM IPBX01 (2 Go RAM · 30 Go disque) | 0.5h |
| T07.2 | Installer FreePBX (ISO AlmaLinux dédié) | 2h |
| T07.3 | Configurer IP statique 172.16.10.6 | 0.5h |
| T07.4 | Créer lignes SIP pour les utilisateurs AD | 2h |
| T07.5 | Installer softphone 3CX sur CLIWIN01 et CLIWIN02 | 1h |
| T07.6 | Tester appel VoIP entre CLIWIN01 et CLIWIN02 | 1h |
| T07.7 | Documenter `voip/` | 1h |

**Messagerie - Zimbra / iRedMail**

| # | Tâche | Durée |
|---|-------|-------|
| T07.8 | Installer Zimbra (ou iRedMail) sur SRVLX01 | +3h |
| T07.9 | Configurer domaine `pharmgreen.fr` | 1h |
| T07.10 | Créer boîtes mail pour les utilisateurs AD | 1h |
| T07.11 | Configurer client mail sur CLIWIN01 et CLIWIN02 | 1h |
| T07.12 | Tester envoi et réception entre 2 clients | 1h |
| T07.13 | Documenter `messagerie/` | 1h |


---

### Sprint 08 - Intégration clients & tests
**Semaine 08 · Durée estimée : 10h**

| Élément | Détail |
|---------|--------|
| **Objectif** | Déployer les postes clients et valider l'ensemble des services |
| **Prérequis** | Sprints 02 à 07 terminés · tous les services opérationnels |
| **Livrable principal** | 2 clients dans le domaine · GPO appliquées · tous les services validés |

| # | Tâche | Durée |
|---|-------|-------|
| T08.1 | Créer VM CLIWIN01 Windows 10 (4 Go RAM · 60 Go) | 1h |
| T08.2 | Créer VM CLIWIN02 Windows 11 (4 Go RAM · 60 Go) | 1h |
| T08.3 | Intégrer CLIWIN01 au domaine `tssr.lan` | 0.5h |
| T08.4 | Intégrer CLIWIN02 au domaine `tssr.lan` | 0.5h |
| T08.5 | Vérifier application des 7 GPO sur les clients | 2h |
| T08.6 | Tester connexion utilisateur AD depuis les clients | 1h |
| T08.7 | Valider DHCP · DNS et résolution de noms | 1h |
| T08.8 | Valider VoIP (appel 3CX) et messagerie Zimbra | 1h |
| T08.9 | Documenter `clients/` | 1h |

---

### Sprint 09 - Objectifs secondaires & redondance AD
**Semaine 09 · Durée estimée : 18h**

| Élément | Détail |
|---------|--------|
| **Objectif** | Réaliser au minimum 50 % des objectifs secondaires |
| **Prérequis** | Sprint 03 terminé · domaine `tssr.lan` stable et validé |
| **Livrable principal** | Minimum 5 objectifs secondaires sur 9 réalisés et documentés |

| # | Tâche | Durée | Priorité |
|---|-------|-------|----------|
| T09.1 | Dossiers partagés + mappage lecteur I: via GPO | 3h | Haute |
| T09.2 | SRVWIN02 Core - AD DS + DNS (réplication) | 3h | Haute |
| T09.3 | SRVWIN03 Core - AD DS + DNS (réplication) | 2h | Haute |
| T09.4 | Répartition des rôles FSMO sur WIN01 / WIN02 / WIN03 | 1h | Haute |
| T09.5 | `GPO-USR-LogonHours-01` (7h30-20h · lun-sam) | 1h | Moyenne |
| T09.6 | Configuration NTP sur SRVWIN01 | 1h | Moyenne |
| T09.7 | `GPO-DOM-WSUS-Update-01` (WSUS via GPO) | 1h | Moyenne |
| T09.8 | Serveur web interne SRVWEB01 (172.16.10.7) | 2h | Moyenne |
| T09.9 | Serveur web externe SRVWEB02 en DMZ (172.16.20.1) | 2h | Moyenne |
| T09.10 | Synchronisation GLPI ↔ Active Directory | 2h | Basse |

---

### Sprint 10 - Finalisation, documentation & présentation
**Semaine 10 · Durée estimée : 18h**

| Élément | Détail |
|---------|--------|
| **Objectif** | Finaliser toute la documentation et préparer la présentation finale |
| **Prérequis** | Tous les objectifs principaux terminés |
| **Livrable principal** | Dépôt GitHub complet · présentation 15-30 min · démo obligatoire |

| # | Tâche | Durée |
|---|-------|-------|
| T10.1 | Compléter tous les fichiers `architecture/` (HLD) | 2h |
| T10.2 | Compléter tous les fichiers `components/` (LLD) | 3h |
| T10.3 | Rédiger `operations/` complet (DEX + SOP) | 2h |
| T10.4 | Mettre à jour README.md de chaque sprint (S01-S10) | 2h |
| T10.5 | Tests end-to-end complets de l'infrastructure | 4h |
| T10.6 | Nettoyer et organiser le dépôt GitHub | 1h |
| T10.7 | Préparer présentation finale (15–30 minutes) | 3h |
| T10.8 | Préparer démo obligatoire (live ou enregistrée) | 1h |

---

## 3. Diagramme de Gantt


<img width="1371" height="487" alt="DIAGRAMME_GANTT_SPRINTS" src="https://github.com/user-attachments/assets/670c1ddc-52dd-4e0f-9142-0d6b62f59781" />


> La documentation est une activité **transversale** à tous les sprints ; elle se rédige en continue.

---

## 4. Dépendances entre sprints

Le déploiement des services suit une logique de dépendances stricte. Un service ne peut pas être déployé si ses prérequis ne sont pas opérationnels.

<img width="528" height="755" alt="DEPENDANCE_SPRINTS" src="https://github.com/user-attachments/assets/4ec41e02-a874-48ea-8234-de71d1366c66" />

---

| Service | Dépend de |
|---------|-----------|
| FW01 pfSense | Aucune dépendance |
| AD DS + DNS + DHCP | FW01 opérationnel |
| GPO | AD DS opérationnel |
| GLPI | DNS fonctionnel · réseau LAN |
| WSUS | AD DS opérationnel · accès internet |
| VoIP FreePBX | Réseau LAN · DNS |
| Zimbra | Réseau LAN · DNS |
| Clients Win10/Win11 | AD DS · DHCP · DNS · GPO |
| Objectifs secondaires | AD DS stable et validé |

---

## 5. Charge de travail estimée

| Sprint | Heures estimées | Type de travail principal |
|--------|----------------|--------------------------|
| Sprint 01 | 11h | Documentation · conception |
| Sprint 02 | 10h | Installation · configuration · tests |
| Sprint 03 | 26h | Installation · configuration · scripting |
| Sprint 04 | 10h | Configuration · tests |
| Sprint 05 | 9h | Installation · configuration |
| Sprint 06 | 9h | Installation · configuration |
| Sprint 07 | 22h | Installation · configuration · tests |
| Sprint 08 | 10h | Tests · intégration |
| Sprint 09 | 18h | Configuration · tests |
| Sprint 10 | 18h | Documentation · présentation |
| **TOTAL** | **~143h** | |

**Répartition par type de tâche :**

| Type | Estimation | Pourcentage |
|------|-----------|-------------|
| Installation et déploiement | ~58h | 41 % |
| Configuration et paramétrage | ~42h | 29 % |
| Documentation | ~28h | 20 % |
| Tests et validation | ~15h | 10 % |

---

## 6. Suivi d'avancement global

> Mettre à jour ce tableau est faite à chaque fin de sprint

| Sprint | Objectif | Taux | Statut | Date fin réelle |
|--------|---------|------|--------|----------------|
| Sprint 01 | Analyse & planification | 78% | En cours | - |
| Sprint 02 | Infrastructure réseau pfSense | 25% | A faire | - |
| Sprint 03 | Domaine AD DS + DNS + DHCP | 0% | A faire | - |
| Sprint 04 | Stratégies GPO & sécurité | 0% | A faire | - |
| Sprint 05 | GLPI & gestion de parc | 0% | A faire | - |
| Sprint 06 | WSUS & mises à jour | 0% | A faire | - |
| Sprint 07 | VoIP FreePBX & messagerie Zimbra | 0% | A faire | - |
| Sprint 08 | Intégration clients & tests | 0% | A faire | - |
| Sprint 09 | Objectifs secondaires & redondance | 0% | A faire | - |
| Sprint 10 | Finalisation, documentation & présentation | 0% | A faire | - |
| **TOTAL PROJET** | | **0%** | ** Non démarré** | - |



---

*Document maintenu par Patrick TAMBWE · Projet TSSR_P3_G7 · Pharmgreen · tssr.lan*
