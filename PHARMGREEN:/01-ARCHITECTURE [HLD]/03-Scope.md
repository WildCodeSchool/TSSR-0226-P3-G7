# scope.md - Périmètre du projet

> **Type** : HLD - High Level Design  
> **Dossier** : `architecture/`  
> **Projet** : TSSR Projet 3 - Build Your Infra  
> **Client** : Pharmgreen - Lyon  
> **Domaine AD** : `tssr.lan`  
> **Dernière mise à jour** : 27/05/2026 
> **Auteur** : Patrick TAMBWE

---

## Sommaire

- [1. Périmètre inclus](#1-périmètre-inclus)
- [2. Périmètre hors scope](#2-périmètre-hors-scope)
- [3. Périmètre temporel](#3-périmètre-temporel)
- [4. Périmètre réseau](#4-périmètre-réseau)
- [5. Périmètre utilisateurs](#5-périmètre-utilisateurs)
- [6. Périmètre matériel](#6-périmètre-matériel)
- [7. Hypothèses et responsabilités](#7-hypothèses-et-responsabilités)
- [8. Risques identifiés](#8-risques-identifiés)

---

## 1. Périmètre inclus

Ce projet couvre la **conception, le déploiement et la documentation complète** des éléments suivants, tous obligatoirement livrés et fonctionnels à l'issue du Sprint 10.

### Infrastructure réseau

| Élément | Description | Obligatoire |
|---------|-------------|-------------|
| Pare-feu pfSense FW01 | 3 interfaces WAN / LAN / DMZ · règles de filtrage · Deny All | Oui |
| Zone LAN 172.16.10.0/24 | Réseau interne - serveurs et clients | Oui |
| Zone DMZ 172.16.20.0/24 | Zone exposée - serveur web externe | Oui |
| Switch LAN virtuel | Distribution interne LAN | Oui |
| Switch DMZ virtuel | Distribution interne DMZ | Oui |

### Services déployés

| Service | Machine | Obligatoire |
|---------|---------|-------------|
| Active Directory DS - domaine tssr.lan | SRVWIN01 | Oui |
| DNS interne - zone tssr.lan | SRVWIN01 | Oui |
| DHCP - plage LAN .10 → .200 | SRVWIN01 | Oui |
| Stratégies GPO (7 obligatoires + 2 au choix) | SRVWIN01 | Oui |
| Gestion de parc GLPI | SRVLX01 | Oui |
| Mises à jour WSUS | SRVWIN04 | Oui |
| Téléphonie VoIP FreePBX | IPBX01 | Oui |
| Messagerie Zimbra / iRedMail | SRVLX01 | Oui |
| Postes clients Windows 10/11 intégrés au domaine | CLIWIN01 / CLIWIN02 | Oui |

### Objectifs secondaires inclus (minimum 50 % requis)

| Objectif secondaire | Machine | Priorité |
|--------------------|---------|----------|
| Dossiers partagés + lecteur réseau I: | SRVWIN01 - 2e disque | Haute |
| Redondance AD DS - 2 DC supplémentaires en Core | SRVWIN02 / SRVWIN03 | Haute |
| Restriction horaire connexions (7h30–20h, lun–sam) | GPO - AD DS | Moyenne |
| Synchronisation NTP | SRVWIN01 | Moyenne |
| WSUS déployé via GPO | SRVWIN04 + GPO | Moyenne |
| Serveur web interne LAN | SRVWEB01 - Debian | Moyenne |
| Serveur web externe DMZ | SRVWEB02 - Debian | Moyenne |
| Synchronisation GLPI ↔ Active Directory | SRVLX01 | Basse |
| Switches virtuels + routeur VyOS | Hyperviseur | Basse |

### Documentation livrée

| Document | Emplacement | Obligatoire |
|----------|-------------|-------------|
| README.md - point d'entrée DAT | `README.md` (racine) | Oui |
| naming.md - nomenclature complète | `naming.md` (racine) | Oui |
| Dossier architecture/ - HLD complet | `architecture/` | Oui |
| Dossier components/ - LLD par brique | `components/` | Oui |
| Dossier operations/ - DEX exploitation | `operations/` | Oui |
| Dossier sprints/ - suivi chronologique | `sprints/` | Oui |

---

## 2. Périmètre hors scope

Les éléments suivants sont **explicitement exclus** de ce projet. Ils ne seront ni déployés, ni documentés, ni pris en charge dans le cadre de cette mission.

### Exclusions techniques

| Élément exclu | Raison de l'exclusion |
|---------------|----------------------|
| Infrastructure physique réelle | Projet pédagogique sur hyperviseur de type 2 uniquement |
| Matériel réseau physique (switches, routeurs) | Virtualisé dans l'hyperviseur |
| Haute disponibilité (HA / clustering) | Hors périmètre TSSR P3 |
| Virtualisation de type 1 (VMware ESXi, Proxmox) | Hyperviseur de type 2 uniquement (VirtualBox / VMware WS) |
| Chiffrement complet des disques (BitLocker, LUKS) | Non requis par le cahier des charges |
| Système de sauvegarde automatisé (Veeam, Bacula) | Non requis par le cahier des charges |
| Supervision réseau (Zabbix, Nagios, PRTG) | Hors périmètre |
| VPN d'accès distant | Aucun besoin de télétravail chez Pharmgreen |
| Infrastructure cloud (Azure AD, AWS, GCP) | Tout en local - pas de cloud hybride |
| Redondance pare-feu (pfSense HA) | Hors périmètre |

### Exclusions organisationnelles

| Élément exclu | Raison de l'exclusion |
|---------------|----------------------|
| Intégration des prestataires externes Kamera | Cahier des charges - prestataires exclus de l'AD DS |
| Gestion des 211 PC portables physiques | Postes virtuels CLIWIN01/02 suffisants pour le projet |
| Migration de la messagerie cloud existante | Nouvelle messagerie interne déployée en parallèle |
| Nomadisme et télétravail | Non requis par Pharmgreen |
| Accès réseau externe hors DMZ | Pas de VPN, pas de RDS |
| Formation des utilisateurs finaux Pharmgreen | Hors périmètre de la mission prestataire |

### Exclusions données

| Élément exclu | Raison de l'exclusion |
|---------------|----------------------|
| Import des téléphones fixes du fichier RH | 12 numéros pour 185 personnes |
| Import des données cloud (drives personnels) | Hors périmètre - données utilisateurs non migrées |
| Données du NAS grand public existant | Hors périmètre - pas de migration de données |

---

## 3. Périmètre temporel

Le projet se déroule sur **10 semaines consécutives**, organisées en **10 sprints hebdomadaires** d'une semaine chacun.

| Sprint | Semaine | Contenu principal | Livrable attendu |
|--------|---------|-------------------|-----------------|
| Sprint 01 | S01 | Analyse, planification, documentation initiale | Arborescence GitHub · README · naming.md · schéma réseau |
| Sprint 02 | S02 | Déploiement FW01 pfSense | FW01 fonctionnel · 3 zones opérationnelles |
| Sprint 03 | S03 | AD DS + DNS + DHCP | Domaine tssr.lan · 208 users importés · DHCP actif |
| Sprint 04 | S04 | Stratégies GPO | 7 GPO obligatoires + 2 personnalisées appliquées |
| Sprint 05 | S05 | GLPI | GLPI accessible web · parc configuré · ticketing actif |
| Sprint 06 | S06 | WSUS | WSUS opérationnel · groupes · MAJ approuvées |
| Sprint 07 | S07 | VoIP FreePBX + Messagerie Zimbra | Appel VoIP testé · messagerie fonctionnelle |
| Sprint 08 | S08 | Clients Windows 10/11 | 2 clients dans le domaine · GPO appliquées · tests validés |
| Sprint 09 | S09 | Objectifs secondaires | Minimum 50 % des objectifs secondaires réalisés |
| Sprint 10 | S10 | Finalisation · documentation · présentation | Dépôt GitHub complet · démo obligatoire |

> Planning détaillé avec durées et prérequis :
> [`../sprints/planning.md`](../sprints/planning.md)

### Critères de fin de projet

Le projet est considéré comme **terminé et livrable** lorsque les conditions suivantes sont toutes remplies.

| Critère | Vérification |
|---------|-------------|
| Tous les objectifs principaux déployés et fonctionnels | Démo en live |
| Minimum 50 % des objectifs secondaires réalisés | Tableau de suivi à jour |
| Dépôt GitHub complet avec DAT + HLD + LLD + DEX + Sprints | Revue du dépôt |
| README.md de chaque sprint rédigé et à jour | Revue du dépôt |
| Présentation finale de 15 à 30 minutes préparée | Soutenance |
| Démo obligatoire réalisée (live ou enregistrée) | Validation formateur |

---

## 4. Périmètre réseau

### Réseaux couverts par ce projet

| Zone | Réseau | Masque | Rôle |
|------|--------|--------|------|
| LAN | 172.16.10.0 | /24 | Réseau interne - serveurs et clients |
| DMZ | 172.16.20.0 | /24 | Zone exposée - serveur web externe |
| WAN | IP box FAI | /24 | Connexion internet via box FAI |

### Réseaux hors scope

| Réseau | Raison |
|--------|--------|
| 172.16.30.0/24 | Réseau Wi-Fi actuel de Pharmgreen - remplacé, non intégré |
| Réseaux externes partenaires | Partenariat en cours non abouti - hors périmètre actuel |
| Réseaux VLAN par département | Objectif secondaire optionnel - non requis |

> Le réseau actuel de Pharmgreen (`172.16.30.0/24`) est remplacé par la nouvelle infrastructure. Il n'y a pas de coexistence des deux réseaux dans le cadre de ce projet pédagogique.

---

## 5. Périmètre utilisateurs

### Utilisateurs intégrés dans l'AD DS

| Population | Nombre | Source | Statut |
|-----------|--------|--------|--------|
| Employés Pharmgreen | 208 | Fichier RH `ListeRHCollaborateurs.xlsx` | À importer |
| Compte prestataire  | 1 | À ajouter manuellement | À créer |
| Comptes administrateurs | 2 minimum | À créer séparément (`adm-prenom.nom`) | À créer |
| **Total comptes AD** | **~211** | | |

### Utilisateurs exclus de l'AD DS

| Population | Nombre | Raison |
|-----------|--------|--------|
| Prestataires Kamera | 3 | Société externe — cahier des charges |
| Autres prestataires ponctuels | Variable | Non intégrés par politique de sécurité |

> Analyse complète du fichier RH et justification des exclusions :
> [`../components/active-directory/ressources/analyse-rh-pharmgreen.xlsx`](../components/active-directory/ressources/analyse-rh-pharmgreen.xlsx)

### Départements couverts

Les **11 départements** de Pharmgreen sont tous représentés dans la structure Active Directory par une OU dédiée et un groupe de sécurité GG correspondant.

| # | Département | OU AD | Groupe GG |
|---|-------------|-------|-----------|
| 01 | Communication | `OU=Communication` | `GG-Communication` |
| 02 | Développement logiciel | `OU=Developpement-Logiciel` | `GG-Developpement-Logiciel` |
| 03 | Direction Financière | `OU=Direction-Financiere` | `GG-Direction-Financiere` |
| 04 | Direction Générale | `OU=Direction-Generale` | `GG-Direction-Generale` |
| 05 | Direction Marketing | `OU=Direction-Marketing` | `GG-Direction-Marketing` |
| 06 | R&D | `OU=R-et-D` | `GG-R-et-D` |
| 07 | RH | `OU=RH` | `GG-RH` |
| 08 | Service Juridique | `OU=Service-Juridique` | `GG-Service-Juridique` |
| 09 | Services Généraux | `OU=Services-Generaux` | `GG-Services-Generaux` |
| 10 | Systèmes d'Information | `OU=Systemes-Information` | `GG-Systemes-Information` |
| 11 | Ventes et Dév. Commercial | `OU=Ventes-et-Dev-Commercial` | `GG-Ventes-Dev-Commercial` |

---

## 6. Périmètre matériel

### Environnement de déploiement

Ce projet est entièrement virtualisé sur un **hyperviseur de type 2** installé sur une machine personnelle. Aucun matériel physique réseau n'est utilisé.

| Élément | Valeur |
|---------|--------|
| Type d'environnement | Virtualisation locale — hyperviseur de type 2 |
| Hyperviseur | VirtualBox |
| Nombre de VMs | 9 minimum (objectifs principaux) · 11 avec objectifs secondaires |
| RAM recommandée machine hôte | 16 Go minimum |
| Stockage recommandé | 200 Go libres minimum |
| Connexion internet | Requise pour les ISOs, WSUS, forwarders DNS |

### VMs dans le périmètre

| VM | OS | RAM | Disque | Zone |
|----|----|----|--------|------|
| FW01 | pfSense | 1 Go | 20 Go | WAN/LAN/DMZ |
| SRVWIN01 | Windows Server 2022 GUI | 4 Go | 60 Go | LAN |
| SRVWIN02 | Windows Server 2022 Core | 2 Go | 40 Go | LAN |
| SRVWIN03 | Windows Server 2022 Core | 2 Go | 40 Go | LAN |
| SRVWIN04 | Windows Server 2022 GUI | 4 Go | 80 Go | LAN |
| SRVLX01 | Debian 12 CLI | 2 Go | 60 Go | LAN |
| IPBX01 | AlmaLinux / FreePBX | 2 Go | 30 Go | LAN |
| SRVWEB02 | Debian 12 CLI | 1 Go | 20 Go | DMZ |
| CLIWIN01 | Windows 10 | 4 Go | 60 Go | LAN |
| CLIWIN02 | Windows 11 | 4 Go | 60 Go | LAN |

> En cas de ressources limitées sur la machine hôte, SRVWIN02 et SRVWIN03 peuvent être déployés en dernier (objectifs secondaires). Les VMs à prioriser sont : FW01 · SRVWIN01 · SRVWIN04 · SRVLX01 · IPBX01 · CLIWIN01 · CLIWIN02.

---

## 7. Hypothèses et responsabilités

### Hypothèses de départ

Ce projet repose sur les hypothèses suivantes, acceptées en début de mission.

| # | Hypothèse | Conséquences si non vérifiée |
|---|-----------|------------------------|
| H01 | La machine hôte dispose d'au moins 16 Go de RAM | Réduction du nombre de VMs actives simultanément |
| H02 | Une connexion internet est disponible sur la machine hôte | Impossible de télécharger les ISOs et les MAJ WSUS |
| H03 | Les licences Windows Server 2022 sont fournies | Blocage du déploiement AD DS / WSUS |
| H04 | Le fichier RH `ListeRHCollaborateurs.xlsx` est la source de vérité | Import AD DS basé sur ce fichier uniquement |
| H05 | Aucune migration de données n'est requise depuis l'ancien NAS | Pas de phase de migration dans ce projet |
| H06 | Le partenariat externe mentionné par Pharmgreen n'aboutit pas pendant le projet | Aucune liaison réseau externe à prévoir |
| H07 | Les 3 prestataires Kamera restent hors périmètre AD DS | Filtre `Société = Pharmgreen` appliqué à l'import |

### Matrice des responsabilités

| Activité | Technicien  | DSI (formateur) |
|----------|-----------------|-----------------|
| Conception de l'architecture | Responsable | Validation |
| Déploiement des VMs | Responsable | - |
| Configuration des services | Responsable | Validation |
| Rédaction de la documentation | Responsable | Validation |
| Validation des livrables de sprint | - | Responsable |
| Définition des exigences | - | Responsable |
| Fourniture des licences Windows | - | Responsable |
| Présentation et démo finale | Responsable | Évaluation |

---

## 8. Risques identifiés

Les risques suivants ont été identifiés en début de projet et font l'objet d'une surveillance tout au long des sprints.

| # | Risque | Probabilité | Impact | Mitigation |
|---|--------|-------------|--------|-----------|
| R01 | Machine hôte insuffisante en RAM | presque élévé | Élevé | Réduire la RAM des VMs au minimum · ne pas allumer toutes les VMs simultanément |
| R02 | Zimbra difficile à installer | Élevée | Moyen | Prévoir à peu près 3h+ · avoir iRedMail en solution de repli |
| R03 | Import des 208 utilisateurs échoue | Faible | Élevé | Tester le script PS sur 5 users avant de lancer l'import complet |
| R04 | Règles pfSense mal configurées bloquent les services | Moyenne | Élevé | Tester chaque règle individuellement avant de passer à la suivante |
| R05 | Promotion DC échoue (nom machine ou réseau) | Faible | Élevé | Nommer et configurer l'IP AVANT la promotion · ne jamais renommer après |
| R06 | Retard sur un sprint bloque les suivants | Moyenne | Moyen | Sprint 09 prévu comme tampon · objectifs secondaires priorisés par impact |
| R07 | Documentation non tenue à jour | Élevée | Moyen | Documenter pendant le déploiement · pas après |
| R08 | Données RH incorrectes (doublons DDN) | Faible | Faible | 4 cas identifiés · signalés dans l'analyse RH · non bloquants |

---

*Document maintenu par Patrick TAMBWE · Projet TSSR_P3_G7 · Pharmgreen · tssr.lan*

