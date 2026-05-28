# naming.md — Nomenclature 

> **Type** : DAT — Documentation d'Architecture Technique  
> **Projet** : TSSR Projet 3 — Build Your Infra  
> **Client** : Pharmgreen — Lyon  
> **Domaine AD** : `tssr.lan`  
> **Dernière mise à jour** : 25/05/2026
> **Auteur** : Patrick TAMBWE

---

## Sommaire

- [1. Domaine Active Directory](#1-domaine-active-directory)
- [2. Machines virtuelles et serveurs](#2-machines-virtuelles-et-serveurs)
- [3. Postes clients](#3-postes-clients)
- [4. Équipements réseau](#4-équipements-réseau)
- [5. Utilisateurs Active Directory](#5-utilisateurs-active-directory)
- [6. Unités d'organisation (OU)](#6-unités-dorganisation-ou)
- [7. Groupes de sécurité](#7-groupes-de-sécurité)
- [8. Stratégies de groupe (GPO)](#8-stratégies-de-groupe-gpo)
- [9. Comptes administrateurs](#9-comptes-administrateurs)
- [10. Messagerie](#10-messagerie)
- [11. Fichiers et dossiers GitHub](#11-fichiers-et-dossiers-github)
- [12. Ressources dans les dossiers](#12-ressources-dans-les-dossiers)
- [13. Gestion des cas particuliers](#13-gestion-des-cas-particuliers)

---

## 1. Domaine Active Directory

| Élément | Valeur | Exemple |
|---------|--------|---------|
| Nom de domaine FQDN | `tssr.lan` | — |
| Nom NetBIOS | `TSSR` | — |
| Niveau fonctionnel | Windows Server 2022 | — |
| DC principal | SRVWIN01 | `SRVWIN01.tssr.lan` |
| DC secondaire | SRVWIN02 | `SRVWIN02.tssr.lan` |
| DC tertiaire | SRVWIN03 | `SRVWIN03.tssr.lan` |


---

## 2. Machines virtuelles et serveurs

### Convention de nommage

```
[TYPE][OS][NUMÉRO]
```

| Segment | Description | Valeurs possibles |
|---------|-------------|-------------------|
| `TYPE` | Type de machine | `SRV` = Serveur · `CLI` = Client · `FW` = Pare-feu · `IPBX` = VoIP |
| `OS` | Système d'exploitation | `WIN` = Windows · `LX` = Linux |
| `NUMÉRO` | Numéro d'ordre à 2 chiffres | `01`, `02`, `03`... |

### Tableau des machines

| Nom machine | Type | OS | Rôle | IP statique | Zone |
|-------------|------|----|------|-------------|------|
| `FW01` | Pare-feu | pfSense (FreeBSD) | Pare-feu périmétrique · routage inter-zones | eth1: 172.16.10.254 · eth2: 172.16.20.254 | WAN/LAN/DMZ |
| `SRVWIN01` | Serveur | Windows Server 2022 GUI | AD DS · DNS · DHCP — DC principal | 172.16.10.1 | LAN |
| `SRVWIN02` | Serveur | Windows Server 2022 Core | AD DS · DNS — DC secondaire | 172.16.10.2 | LAN |
| `SRVWIN03` | Serveur | Windows Server 2022 Core | AD DS · DNS — DC tertiaire | 172.16.10.3 | LAN |
| `SRVWIN04` | Serveur | Windows Server 2022 GUI | WSUS | 172.16.10.4 | LAN |
| `SRVLX01` | Serveur | Debian 13 CLI | GLPI · Messagerie Zimbra | 172.16.10.5 | LAN |
| `IPBX01` | Serveur VoIP | AlmaLinux / FreePBX | Téléphonie VoIP | 172.16.10.6 | LAN |
| `SRVWEB01` | Serveur | Debian 13 CLI | Serveur web interne | 172.16.10.7 | LAN |
| `SRVWEB02` | Serveur | Debian 13 CLI | Serveur web externe | 172.16.20.1 | DMZ |
| `CLIWIN01` | Client | Windows 11 | Poste utilisateur | DHCP .10→.200 | LAN |
| `CLIWIN02` | Client | Windows 11 | Poste utilisateur | DHCP .10→.200 | LAN |

### Règles complémentaires

- Le nom machine est défini **avant** toute installation — il ne doit pas être modifié après jonction au domaine
- Le nom est saisi en **MAJUSCULES** sur la machine elle-même
- Le nom est toujours en **minuscules** dans la documentation Markdown
- Numérotation séquentielle à partir de `01` — jamais de `0` seul

---

## 3. Postes clients

### Convention

```
CLI/OS/NUMÉRO
```

| Nom | OS | Usage | Remarque |
|-----|----|-------|----------|
| `CLIWIN01` | Windows 10 | Poste client 1 — membre du domaine tssr.lan | Softphone 3CX installé |
| `CLIWIN02` | Windows 11 | Poste client 2 — membre du domaine tssr.lan | Softphone 3CX installé |

> Les 211 postes Pharmgreen (PC portables hétérogènes) suivront la même convention lors d'une intégration future au domaine.

---

## 4. Équipements réseau

| Nom | Type | Rôle | Remarque |
|-----|------|------|----------|
| `FW01` | Pare-feu virtuel | pfSense — 3 interfaces | Unique pare-feu de l'infrastructure |
| `SW-LAN` | Switch virtuel | Distribution LAN 172.16.10.0/24 | Créé dans l'hyperviseur |
| `SW-DMZ` | Switch virtuel | Distribution DMZ 172.16.20.0/24 | Créé dans l'hyperviseur |

---

## 5. Utilisateurs Active Directory

### Convention de nommage du login

```
prenom.nom
```

| Règle | Description | Exemple |
|-------|-------------|---------|
| Format | `prenom.nom` tout en minuscules | `valentina.ferrari` |
| Accents | Remplacés par leur équivalent sans accent | `éèê` → `e` · `àâ` → `a` · `ç` → `c` |
| Espaces | Remplacés par un tiret `-` dans le nom | `Da Silva` → `da-silva` |
| Tirets existants | Conservés tels quels | `Gagnon-Leclercq` → `gagnon-leclercq` |
| Homonymes | Suffixe numérique sur le nom | `hugo.durand` → `hugo.durand2` |
| Casse | Toujours en minuscules | jamais `Valentina.Ferrari` |

### Exemples concrets (extraits du fichier RH)

| Prénom | Nom | Login AD | Email |
|--------|-----|----------|-------|
| Valentina | Ferrari | `valentina.ferrari` | `valentina.ferrari@pharmgreen.fr` |
| Kelly | Arnaud | `kelly.arnaud` | `kelly.arnaud@pharmgreen.fr` |
| Léa | Gagnon-Leclercq | `lea.gagnon-leclercq` | `lea.gagnon-leclercq@pharmgreen.fr` |
| Estelle | Dubois--Melun | `estelle.dubois-melun` | `estelle.dubois-melun@pharmgreen.fr` |
| Thomas-Ambroise | Gauthier | `thomas-ambroise.gauthier` | `thomas-ambroise.gauthier@pharmgreen.fr` |
| Isabella | Da Silva | `isabella.da-silva` | `isabella.da-silva@pharmgreen.fr` |

### Gestion des homonymes

Dans le cas où deux personnes auraient exactement le même prénom et le même nom (aucun cas détecté dans le fichier RH actuel), la règle est la suivante :

| Cas | Login |
|-----|-------|
| Premier arrivé | `prenom.nom` |
| Deuxième | `prenom.nom2` |
| Troisième | `prenom.nom3` |

### Attributs AD obligatoires à renseigner

| Attribut AD | Source fichier RH | Exemple |
|-------------|-------------------|---------|
| `SamAccountName` | Colonne Prénom + Nom | `valentina.ferrari` |
| `UserPrincipalName` | Login + domaine | `valentina.ferrari@tssr.lan` |
| `GivenName` | Colonne Prénom | `Valentina` |
| `Surname` | Colonne Nom | `Ferrari` |
| `DisplayName` | Prénom + Nom | `Valentina Ferrari` |
| `EmailAddress` | Convention messagerie | `valentina.ferrari@pharmgreen.fr` |
| `Department` | Colonne Département | `Communication` |
| `Title` | Colonne Fonction | `Directrice communication` |
| `Manager` | Colonnes Manager prénom + nom | `kelly.arnaud` |
| `MobilePhone` | Colonne Téléphone portable | `0666052403` |


---

## 6. Unités d'organisation (OU)

### Structure hiérarchique

```
tssr.lan
└── OU=Pharmgreen                       ← Conteneur racine entreprise
    ├── OU=_Utilisateurs                ← Tous les comptes utilisateurs
    │   ├── OU=Communication
    │   │   ├── OU=Gestion-des-marques
    │   │   ├── OU=Publicite
    │   │   ├── OU=Relation-Medias
    │   │   └── OU=Relation-Publique-et-Presse
    │   ├── OU=Developpement-Logiciel
    │   │   ├── OU=Analyse-et-Conception
    │   │   ├── OU=Developpement
    │   │   └── OU=Tests-et-Qualite
    │   ├── OU=Direction-Financiere
    │   │   ├── OU=Comptabilite
    │   │   ├── OU=Controle-de-Gestion
    │   │   └── OU=Finance
    │   ├── OU=Direction-Generale
    │   ├── OU=Direction-Marketing
    │   │   ├── OU=Marketing-Digital
    │   │   ├── OU=Marketing-Operationnel
    │   │   ├── OU=Marketing-Produit
    │   │   └── OU=Marketing-Strategique
    │   ├── OU=R-et-D
    │   │   ├── OU=Innovation-et-Strategie
    │   │   └── OU=Laboratoire
    │   ├── OU=RH
    │   │   ├── OU=Formation
    │   │   ├── OU=Gestion-des-Performances
    │   │   ├── OU=Recrutement
    │   │   └── OU=Sante-et-Securite
    │   ├── OU=Service-Juridique
    │   │   ├── OU=Contentieux
    │   │   ├── OU=Contrats
    │   │   └── OU=Propriete-Intellectuelle
    │   ├── OU=Services-Generaux
    │   │   ├── OU=Gestion-Immobiliere
    │   │   └── OU=Logistique
    │   ├── OU=Systemes-Information
    │   │   ├── OU=Data
    │   │   └── OU=Developpement-Logiciel-SI
    │   └── OU=Ventes-et-Dev-Commercial
    │       ├── OU=ADV
    │       ├── OU=B2B
    │       ├── OU=B2C
    │       ├── OU=Developpement-International
    │       ├── OU=Grands-Comptes
    │       ├── OU=Service-Achat
    │       └── OU=Service-Client
    ├── OU=_Ordinateurs                 ← Postes clients membres du domaine
    ├── OU=_Groupes                     ← Groupes de sécurité GG et GDL
    ├── OU=_Admins                      ← Comptes administrateurs IT
    └── OU=_Serveurs                    ← Objets ordinateurs serveurs
```

### Règles de nommage des OU

| Règle | Description | Exemple |
|-------|-------------|---------|
| Casse | PascalCase avec tirets | `OU=Direction-Financiere` |
| Accents | Supprimés | `Générale` → `Generale` |
| Espaces | Remplacés par tirets `-` | `Gestion des marques` → `Gestion-des-marques` |
| Préfixe `_` | Pour les OU de structure | `_Utilisateurs` · `_Groupes` |
| Esperluette `&` | Remplacée par `-et-` | `R&D` → `R-et-D` |

---

## 7. Groupes de sécurité

### Logique AGDLP appliquée

```
Accounts → Global groups → Domain Local groups → Permissions
Comptes utilisateurs → GG → GDL → Ressource (dossier partagé, imprimante...)
```

### Convention de nommage

```
[TYPE]-[Département/Service]-[Droit optionnel]
```

| Type | Signification | Usage |
|------|--------------|-------|
| `GG` | Groupe Global | Regroupe les comptes utilisateurs d'un département |
| `GDL` | Groupe de Domaine Local | Donne accès à une ressource spécifique |

### Groupes globaux (GG) — un par département

| Nom du groupe | Département concerné |
|---------------|---------------------|
| `GG-Communication` | Communication |
| `GG-Developpement-Logiciel` | Développement logiciel |
| `GG-Direction-Financiere` | Direction Financière |
| `GG-Direction-Generale` | Direction Générale |
| `GG-Direction-Marketing` | Direction Marketing |
| `GG-R-et-D` | R&D |
| `GG-RH` | Ressources Humaines |
| `GG-Service-Juridique` | Service Juridique |
| `GG-Services-Generaux` | Services Généraux |
| `GG-Systemes-Information` | Systèmes d'Information |
| `GG-Ventes-Dev-Commercial` | Ventes et Développement Commercial |

### Groupes de domaine local (GDL) — accès aux ressources

| Nom du groupe | Ressource concernée | Droit |
|---------------|---------------------|-------|
| `GDL-Partages-Lecture` | Dossiers partagés | Lecture seule |
| `GDL-Partages-Ecriture` | Dossiers partagés | Lecture + Écriture |
| `GDL-GLPI-Utilisateurs` | Application GLPI | Accès utilisateur |
| `GDL-GLPI-Admins` | Application GLPI | Accès administrateur |
| `GDL-WSUS-Serveurs` | Groupe WSUS | Machines serveurs |
| `GDL-WSUS-Clients` | Groupe WSUS | Machines clientes |
| `GDL-Individuel-[login]` | Dossier individuel lecteur I: | Accès exclusif |

---

## 8. Stratégies de groupe (GPO)

### Convention de nommage

```
GPO-[CIBLE]-[FONCTION]-[ID]
```

| Segment | Description | Valeurs |
|---------|-------------|---------|
| `GPO` | Préfixe fixe | Toujours `GPO` |
| `CIBLE` | À qui s'applique la GPO | `DOM` = domaine entier · `USR` = utilisateurs · `SRV` = serveurs · `CLI` = clients |
| `FONCTION` | Ce que fait la GPO | En PascalCase, sans espace |
| `ID` | Numéro d'ordre | `01`, `02`, `03`... |

### Liste complète des GPO du projet

| Nom GPO | Cible | Fonction | Lié à | Obligatoire |
|---------|-------|----------|-------|-------------|
| `GPO-DOM-PasswordPolicy-01` | Domaine | Politique de mots de passe (complexité, longueur min. 8) | `OU=Pharmgreen` | ✅ Oui |
| `GPO-DOM-AccountLockout-01` | Domaine | Verrouillage compte après 5 erreurs | `OU=Pharmgreen` | ✅ Oui |
| `GPO-DOM-ControlPanel-Block-01` | Utilisateurs | Blocage panneau de configuration | `OU=_Utilisateurs` | ✅ Oui |
| `GPO-DOM-LocalAdmin-01` | Ordinateurs | Compte domaine = admin local machines | `OU=_Ordinateurs` | ✅ Oui |
| `GPO-DOM-PowerShell-Security-01` | Domaine | Politique de sécurité PowerShell | `OU=Pharmgreen` | ✅ Oui |
| `GPO-DOM-Custom-01` | Domaine | GPO personnalisée au choix | `OU=Pharmgreen` | ✅ Oui |
| `GPO-DOM-Custom-02` | Domaine | GPO personnalisée au choix | `OU=Pharmgreen` | ✅ Oui |
| `GPO-USR-LogonHours-01` | Utilisateurs | Restriction horaire 7h30–20h lun–sam | `OU=_Utilisateurs` | ⭕ Secondaire |
| `GPO-DOM-WSUS-Update-01` | Ordinateurs | Déploiement WSUS via GPO | `OU=_Ordinateurs` | ⭕ Secondaire |
| `GPO-USR-DriveMapping-I-01` | Utilisateurs | Mappage lecteur réseau I: | `OU=_Utilisateurs` | ⭕ Secondaire |

---

## 9. Comptes administrateurs

### Convention

Les comptes administrateurs sont **séparés** des comptes utilisateurs standards. Un administrateur dispose de deux comptes distincts :

| Type | Format | Exemple | Usage |
|------|--------|---------|-------|
| Compte utilisateur standard | `prenom.nom` | `valentina.ferrari` | Travail quotidien |
| Compte administrateur dédié | `adm-prenom.nom` | `adm-valentina.ferrari` | Administration système uniquement |

### Comptes système par défaut

| Compte | Machine | Mot de passe par défaut | À changer |
|--------|---------|------------------------|-----------|
| `Administrator` | SRVWIN01/02/03/04 | `Azerty1*` | ✅ Oui — dès l'installation |
| `admin` | FW01 pfSense | `pfsense` | ✅ Oui — dès la 1ère connexion |
| `root` | SRVLX01 | `Azerty1*` | ✅ Oui — dès l'installation |
| `wilder` | Toutes VMs | `Azerty1*` | ✅ Oui — à la mise en production |

> ⚠️ Les mots de passe par défaut sont uniquement pour la phase de déploiement. Ils doivent être changés avant toute mise en production.

---

## 10. Messagerie

### Convention des adresses mail

```
prenom.nom@pharmgreen.fr
```

| Règle | Description | Exemple |
|-------|-------------|---------|
| Format | Identique au login AD | `valentina.ferrari@pharmgreen.fr` |
| Domaine | Toujours `@pharmgreen.fr` | Cohérence avec messagerie cloud existante |
| Accents | Supprimés (même règle que login AD) | `lea.gagnon-leclercq@pharmgreen.fr` |

> Le format `prenom.nom@pharmgreen.fr` était déjà utilisé par la messagerie cloud de Pharmgreen avant le projet — la convention est donc maintenue pour assurer la continuité.

---

## 11. Fichiers et dossiers GitHub

### Règles générales

| Règle | Description | Exemple correct | Exemple incorrect |
|-------|-------------|-----------------|-------------------|
| Langue | Toujours en anglais | `network.md` | `reseau.md` |
| Espaces | Interdits — utiliser `-` ou `_` | `ip-configuration.md` | `ip configuration.md` |
| Accents | Interdits | `security.md` | `sécurité.md` |
| Majuscules | Uniquement pour `README.md` et `SOP/` | `README.md` | `Readme.md` |
| Extension | Toujours `.md` pour la documentation | `overview.md` | `overview.txt` |

### Convention par type de fichier

| Type de fichier | Convention | Exemple |
|----------------|-----------|---------|
| Documentation principale | `nom-descriptif.md` | `ip_configuration.md` |
| Fichier README | `README.md` (majuscules) | `README.md` |
| Scripts PowerShell | `action-cible.ps1` | `import-users.ps1` |
| Scripts Bash | `action-cible.sh` | `install-glpi.sh` |
| Captures d'écran | `machine-action-detail.png` | `fw01-regle-lan-wan.png` |
| Schémas | `schema-sujet.svg` | `schema-reseau-pharmgreen.svg` |
| Fichiers de config | `service-config.conf` ou `.xml` | `pfsense-rules-export.xml` |

---

## 12. Ressources dans les dossiers

Chaque dossier de brique dans `components/` contient un sous-dossier `ressources/` qui suit ces conventions :

### Nommage des captures d'écran

```
[machine]-[action]-[detail].png
```

| Exemple correct | Explication |
|-----------------|-------------|
| `fw01-interface-wan-config.png` | FW01 · configuration interface WAN |
| `srvwin01-ad-structure-ou.png` | SRVWIN01 · structure OU Active Directory |
| `srvwin01-dhcp-plage-lan.png` | SRVWIN01 · plage DHCP LAN |
| `glpi-interface-parc.png` | GLPI · interface gestion de parc |
| `gpo-password-policy-settings.png` | GPO · paramètres politique mot de passe |

### Nommage des scripts

| Exemple correct | Explication |
|-----------------|-------------|
| `import-users.ps1` | Script PS import utilisateurs AD |
| `create-ou-structure.ps1` | Script PS création structure OU |
| `install-glpi.sh` | Script Bash installation GLPI |
| `configure-wsus.ps1` | Script PS configuration WSUS |

> ❌ **À ne jamais faire** : `image1.png`, `capture écran 3.png`, `script final VRAI.ps1`, `copie de backup.md`

---

## 13. Gestion des cas particuliers

### Caractères spéciaux dans les noms

| Caractère | Remplacement | Exemple |
|-----------|-------------|---------|
| Accent aigu `é è ê ë` | `e` | `Générale` → `Generale` |
| Accent grave/circonflexe `à â` | `a` | `Château` → `Chateau` |
| Cédille `ç` | `c` | `François` → `Francois` |
| Accent `î ï` | `i` | `Île` → `Ile` |
| Accent `ô ö` | `o` | `Côté` → `Cote` |
| Accent `ù û ü` | `u` | `Où` → `Ou` |
| Espace | `-` (tiret) dans noms de fichiers | `Direction Marketing` → `Direction-Marketing` |
| Esperluette `&` | `-et-` dans les OU | `R&D` → `R-et-D` |
| Double tiret `--` | Tiret simple `-` | `Dubois--Melun` → `dubois-melun` |

### Prestataires externes

Les 3 prestataires de la société **Kamera** (Ariane Kaine, Jean Maire, Jean-Pascal Millith) travaillent dans le département Communication mais **ne sont pas intégrés** dans l'AD DS de Pharmgreen, conformément aux exigences du projet et aux bonnes pratiques de sécurité.

> 📄 Détail complet → [`components/active-directory/ressources/analyse-rh-pharmgreen.xlsx`](components/active-directory/ressources/analyse-rh-pharmgreen.xlsx)

---

*Document maintenu par [Ton nom] · Projet TSSR P3 · Pharmgreen · tssr.lan*
