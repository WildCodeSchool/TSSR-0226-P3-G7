# INSTALLATION : WSUS (SRVWIN04)

> **Type** : LLD - Low Level Design  
> **Dossier** : `components/fw01/`  
> **Sprint** : Sprint 06  
> **Statut** : Partielement Terminé  

---

## Sommaire

- [1. Prérequis](#1-prérequis)
- [2. Création ou Origine de la VM dans Virtualbox](#2-creation-ou-origine-de-la-vm-dans-virtualbox)
- [3. Installation du rôle WSUS](#3-installation-du-role-wsus)
- [4. Premier lancement de la console WSUS](#4-premier-lancement-de-la-console-wsus)
- [5. Création des groupes de machines](#5-creation-des-groupes-de-machines)
- [6. Approbation automatique des mises à jour](#6-approbation-automatique-des-mises-a-jour)
- [7. Déploiement coté client](#7-deploiement-cote-client)
- [8. Vérifivation finale de installation](#8-verification-finale-de-installation)

---




------------------------------------------------------------------------------------------------------------------





## 1 Prérequis

## 2. Création ou Origine de la VM dans Virtualbox 

Contrairement aux autres serveurs du projet, **SRVWIN04 a été créée par clonage** d'un template Windows Server existant (plutôt que par installation complète depuis une ISO), puis reconfigurée aux standards du projet.

| Champ | Valeur |
|---|---|
| Nom | SRVWIN04 |
| Système | Windows Server 2022 *(hérité du template cloné)* |
| Réseau | Adaptateur 1 -> Réseau interne -> `LAN-Pharmgreen` |

### 2.1 Reconfiguration post-clonage

Après clonage, la VM a été renommée et rejointe au domaine :

```powershell
Rename-Computer -NewName "SRVWIN04" -Restart
```

Jonction au domaine :
```powershell
$cred = Get-Credential TSSR\Administrateur
Add-Computer -DomainName "tssr.lan" -Credential $cred -Restart
```

### 2.2 IP statique

| Paramètre | Valeur |
|---|---|
| Adresse IP | 172.16.10.4 /24 |
| Passerelle | 172.16.10.254 (FW01) |
| DNS | 172.16.10.1 (SRVWIN01) |

### 2.3 Vérification de la jonction au domaine

```powershell
Get-ComputerInfo | Select CsDomain, CsDomainRole
hostname
```

Résultat attendu : `CsDomain = tssr.lan`, `CsDomainRole = MemberServer`, `hostname = SRVWIN04`.

---

## 3 Installation du rôle WSUS

### 3.1 Créer le dossier de stockage du contenu

```powershell
New-Item -Path "C:\WSUS" -ItemType Directory
```

> Le chemin dépend de la configuration disque de la VM : utiliser `C:\WSUS` en l'absence de disque secondaire dédié (`D:\`).

### 3.2 Installer le rôle

```powershell
Install-WindowsFeature -Name UpdateServices -IncludeManagementTools
```

Vérifier le résultat : `Success : True`.

### 3.3 Post-installation : initialiser le dossier de contenu

```powershell
cd "C:\Program Files\Update Services\Tools"
.\wsusutil.exe postinstall CONTENT_DIR=C:\WSUS
```

> Le chemin passé à `CONTENT_DIR` doit correspondre exactement au disque réellement disponible sur la VM - une erreur de lettre de lecteur (ex. `D:\WSUS` sur une machine sans disque D:) provoque l'échec `The device is not ready`.

Cette étape peut prendre plusieurs minutes (création de la base de données interne WSUS/WID).

---

## 4 Premier lancement de la console WSUS

Au premier lancement (**Outils -> Windows Server Update Services**), l'assistant de configuration se déclenche :

1. **Before You Begin** -> Suivant
2. **Join the Microsoft Update Improvement Program** -> décoché -> Suivant
3. **Choose Upstream Server** -> **Synchronize from Microsoft Update** -> Suivant
4. **Specify Proxy Server** -> aucun proxy nécessaire (accès direct via NAT FW01) -> Suivant
5. **Connect to Upstream Server** -> **Start Connecting**
6. **Choose Languages** -> Français, Anglais (selon besoin) -> Suivant
7. **Choose Products** -> sélectionner les produits Windows Server / Windows 11 concernés par le parc Pharmgreen -> Suivant
8. **Choose Classifications** -> Critical Updates, Security Updates *(a minima)* -> Suivant
9. **Set Sync Schedule** -> synchronisation manuelle ou planifiée quotidienne -> Suivant
10. **Finished** -> décocher **"Begin initial synchronization"** si l'on préfère lancer la synchro manuellement après coup

---

## 5 Création des groupes de machines

**Console WSUS -> Computers -> All Computers -> Add Computer Group...**

| Groupe | Usage |
|---|---|
| Serveurs | Machines serveurs Windows Server |
| Postes clients | Postes Windows 11 (CLIWIN01, CLIWIN02) |

Procédure : clic droit sur **All Computers -> Add Computer Group... -> nom du groupe -> Add**.

---

## 6 Approbation automatique des mises à jour

**Console WSUS -> Options -> Automatic Approvals -> Update Rules** :

Règle **Default Automatic Approval Rule** :
- Déclencheur : **Critical Updates, Security Updates**
- Cible : **all computers**

> Cette étape peut être bloquée (`Cannot save configuration because the server is synchronizing`) si une synchronisation est active ou en tentative répétée , WSUS verrouille certains paramètres dans ce cas. Relancer la sauvegarde après l'échec d'une tentative de synchronisation, moment où le verrou se libère temporairement.

---

## 7 Déploiement côté client 
(une fois la synchronisation opérationnelle)

Sur un poste client déjà membre du domaine (ex. CLIWIN01) :

```cmd
gpupdate /force
wuauclt /detectnow
```

Vérifier dans la console WSUS que le poste apparaît dans **Unassigned Computers**, puis l'assigner manuellement au groupe **Postes clients**.


## 8 Vérification finale de installation

- [ ] Rôle WSUS installé et console accessible via Gestionnaire de serveur -> Outils -> Windows Server Update Services
- [ ] SRVWIN04 correctement nommée et jointe au domaine tssr.lan
- [ ] IP statique 172.16.10.4 confirmée
- [ ] Dossier de contenu C:\WSUS créé et référencé dans la post-install

Pour la configuration fonctionnelle (groupes de machines, règles d'approbation) et le suivi de l'incident de synchronisation sont détaillés dans **`CONFIGURATION.md`**.
