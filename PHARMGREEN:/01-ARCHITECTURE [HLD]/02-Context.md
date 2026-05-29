# context.md - Contexte et besoins du projet

> **Type** : HLD - High Level Design  
> **Dossier** : `architecture/`  
> **Projet** : TSSR Projet 3 - Build Your Infra  
> **Client** : Pharmgreen - Lyon  
> **Dernière mise à jour** : 28/05/2026  
> **Auteur** : Patrick TAMBWE

---

## Sommaire

- [1. Contexte métier](#1-contexte-métier)
- [2. Présentation de Pharmgreen](#2-présentation-de-pharmgreen)
- [3. Situation initiale et état de l'existant](#3-situation-initiale-et-état-de-lexistant)
- [4. Besoins fonctionnels principaux](#4-besoins-fonctionnels-principaux)
- [5. Besoins fonctionnels secondaires](#5-besoins-fonctionnels-secondaires)
- [6. Contraintes identifiées](#6-contraintes-identifiées)

---

## 1. Contexte métier

Notre équipe a été mandatée par le DSI de Pharmgreen pour **concevoir et déployer une infrastructure réseau professionnelle complète**, en remplacement de l'environnement actuel non structuré et non sécurisé.

Ce projet s'inscrit dans une démarche de **modernisation et de professionnalisation** du système d'information de l'entreprise, qui connaît une croissance soutenue et dont les pratiques informatiques actuelles ne répondent plus aux exigences de sécurité, de fiabilité et de gestion d'une structure de 211 personnes.

---

## 2. Présentation de Pharmgreen

| Élément | Valeur |
|---------|--------|
| Secteur d'activité | Fabrication de dispositifs médicaux d'origine végétale |
| Localisation | Lyon |
| Effectif | 211 collaborateurs |
| Nombre de départements | 11 |
| Messagerie actuelle | Cloud - `prenom.nom@pharmgreen.fr` |
| Fondateur | Originaire de Lyon |

Pharmgreen se positionne comme un **pionnier dans le domaine de la santé naturelle**, en proposant des dispositifs médicaux innovants d'origine végétale. Son effectif est réparti dans 11 départements couvrant la direction, la R&D, le développement logiciel, la communication, le marketing, les ressources humaines, le service juridique, les services généraux, la finance, les systèmes d'information et les ventes.

> Un partenariat externe est en cours de négociation et pourrait aboutir dans les prochains mois. Cette éventualité a été prise en compte dans la conception de l'infrastructure (plages IP disponibles en DMZ).

---

## 3. Situation initiale et état de l'existant

### Réseau

L'ensemble des utilisateurs accède à internet via une **simple box FAI** complétée par des répéteurs Wi-Fi, sur un réseau unique non segmenté en `172.16.30.0/24`. Il n'existe aucun équipement réseau administrable, aucun pare-feu dédié et aucune séparation entre les flux.

### Postes de travail

Les 211 collaborateurs utilisent des **PC portables de marques hétérogènes**, tous configurés en **workgroup** avec des comptes locaux. Le turnover important des stagiaires, alternants et CDD a conduit à une réutilisation systématique des mots de passe, exposant l'entreprise à des risques de sécurité majeurs.

### Identité et sécurité

Il n'existe **aucun contrôle d'identité centralisé**. Pas de domaine Active Directory, pas de politique de mots de passe, pas de journalisation des accès. Chaque poste est indépendant et géré localement.

### Stockage et sauvegarde

Les données des encadrants sont stockées sur un **NAS grand public sans redondance**, accessible uniquement aux responsables et directeurs. Les autres collaborateurs utilisent des **drives cloud personnels**. Les sauvegardes sont effectuées ponctuellement, sans durée de rétention définie.

### Téléphonie

La téléphonie est **hétérogène** : mélange de téléphones fixes et mobiles sans système unifié. Aucune solution VoIP d'entreprise n'est en place.

### Messagerie

La messagerie est **hébergée en cloud** au format `prenom.nom@pharmgreen.fr`. Elle est accessible uniquement via webmail en dehors du site. Aucune messagerie interne n'existe.

### Tableau de synthèse de l'existant

| Domaine | Situation actuelle | Niveau de risque |
|---------|-------------------|-----------------|
| Réseau | Box FAI + Wi-Fi · réseau unique non segmenté | Critique |
| Identité | Workgroup · comptes locaux · mots de passe partagés | Critique |
| Sécurité | Aucun pare-feu · aucune politique · aucune journalisation | Critique |
| Stockage | NAS grand public sans redondance . drives cloud personnels | Élevé |
| Téléphonie | Hétérogène · pas de VoIP unifiée | Moyen |
| Messagerie | Cloud externe · webmail uniquement | Moyen |
| Sauvegarde | Ponctuelle · pas de rétention | Élevé |
| Postes | PC portables hétérogènes en workgroup | Critique |

---

## 4. Besoins fonctionnels principaux

Ces besoins sont **obligatoires** et constituent le socle minimal du livrable final.

| # | Besoin | Solution retenue |
|---|--------|-----------------|
| B01 | Sécuriser le périmètre réseau avec zones distinctes | Pare-feu pfSense FW01 - zones WAN / LAN / DMZ |
| B02 | Centraliser la gestion des identités et des accès | Active Directory DS - domaine `tssr.lan` |
| B03 | Assurer la résolution de noms interne | Service DNS - zone directe `tssr.lan` |
| B04 | Distribuer automatiquement les adresses IP | Service DHCP - plage `172.16.10.10 → .200` |
| B05 | Appliquer des politiques de sécurité sur les postes | Stratégies GPO - 7 obligatoires + 2 au choix |
| B06 | Gérer le parc informatique et les incidents | GLPI - gestion de parc + ticketing |
| B07 | Centraliser et contrôler les mises à jour Windows | WSUS - groupes de machines + MAJ sécurité |
| B08 | Unifier la téléphonie en solution VoIP interne | FreePBX + softphone 3CX |
| B09 | Fournir une messagerie professionnelle interne | Zimbra ou iRedMail - domaine `pharmgreen.fr` |
| B10 | Intégrer les postes clients au domaine | CLIWIN01 (Win10) + CLIWIN02 (Win11) |

---

## 5. Besoins fonctionnels secondaires

Rappel: Ces besoins sont **optionnels** mais au moins 50 % doivent être réalisés.

| # | Besoin | Solution retenue |
|---|--------|-----------------|
| S01 | Dossiers partagés avec lecteur réseau mappé (I:) | 2e disque SRVWIN01 + GPO mappage |
| S02 | Redondance du contrôleur de domaine | SRVWIN02 + SRVWIN03 en Windows Server Core |
| S03 | Restreindre les connexions en dehors des horaires de travail | GPO restriction horaire 7h30-20h lun-sam |
| S04 | Synchroniser l'heure sur tous les équipements | NTP sur SRVWIN01 |
| S05 | Déployer WSUS automatiquement via GPO | GPO-DOM-WSUS-Update-01 |
| S06 | Publier un site web interne accessible sur le LAN | SRVWEB01 - Apache/Nginx |
| S07 | Publier un site web externe accessible depuis internet | SRVWEB02 en DMZ - Apache/Nginx |
| S08 | Synchroniser GLPI avec l'Active Directory | Synchronisation LDAP/AD dans GLPI |
| S09 | Segmenter le LAN avec des switches virtuels et un routeur | VyOS + switches virtuels dans l'hyperviseur |

---

## 6. Contraintes identifiées

| Contrainte | Type | Impact sur le projet |
|-----------|------|---------------------|
| Environnement virtualisé uniquement | Technique | Performances limitées par la RAM de la machine hôte |
| Licences Windows Server fournies par l'école | Organisationnelle | Dépendance externe pour le démarrage |
| 10 semaines - 1 sprint par semaine | Temporelle | Priorisation stricte des objectifs principaux |
| Fichier RH contient des données fictives (tél. fixe) | Données | Champ téléphone fixe non importé dans l'AD |
| 3 prestataires Kamera à exclure de l'AD DS | Organisationnelle | Filtre `Société = Pharmgreen` dans le script d'import |
| Nom du domaine AD non modifiable après promotion | Technique | `tssr.lan` défini définitivement dès le Sprint 03 |
| Zimbra chronophage à l'installation | Technique | 5h+ prévues — iRedMail en solution de repli |

---

*Document maintenu par Patrick TAMBWE · Projet TSSR_P3_G7 · Pharmgreen · tssr.lan*

