# Intégration clients

---

## 1. Installation des VM CLIWIN01 et CLIWIN02

### 1.1 Caractéristiques des VM

| Paramètre | CLIWIN01 | CLIWIN02 |
|---|---|---|
| Système | Windows 11 (ISO) | Windows 11 (ISO) |
| RAM | 4 Go | 4 Go |
| Disque | 60 Go (VDI) | 60 Go (VDI) |
| Réseau | Adaptateur 1 -> Réseau interne -> `LAN-Pharmgreen` | Adaptateur 1 -> Réseau interne -> `LAN-Pharmgreen` |
| Nom de machine | CLIWIN01 | CLIWIN02 |

> L'ISO Windows 10 n'étant pas disponible, les deux postes clients ont été installés avec l'ISO Windows 11.

### 1.2 Contournement du compte Microsoft obligatoire

L'installation standard de Windows 11 impose la création d'un compte Microsoft en ligne. Pour rester cohérent avec un poste destiné à un compte de domaine, ce contournement a été appliqué à l'écran de configuration réseau (OOBE) :

1. À l'écran demandant la connexion Wi-Fi/réseau, ouvrir l'invite de commandes : `Maj + F10`
2. Saisir :
```
oobe\bypassnro
```
3. La machine redémarre et propose alors la création d'un **compte local**

### 1.3 Compte local temporaire

Un compte administrateur local temporaire (`wilder` / `Azerty1*`) a été créé à l'issue de l'installation, le temps de procéder à la jonction au domaine.

---

## 2. Configuration réseau (DHCP automatique)

Aucune configuration IP manuelle n'a été appliquée : les deux postes obtiennent leur adresse via le serveur DHCP de SRVWIN01 (scope LAN-Pharmgreen, 172.16.10.10 -> 200).

### 2.1 Vérification de l'attribution

```powershell
ipconfig /all | Select-String "IPv4|DHCP|Bail|Lease|Serveur DHCP|DHCP Server"
```

Résultat obtenu :

| Poste | Adresse IPv4 | Serveur DHCP |
|---|---|---|
| CLIWIN01 | 172.16.10.12 | 172.16.10.1 |
| CLIWIN02 | 172.16.10.11 | 172.16.10.1 |

---

## 3. Jonction au domaine tssr.lan

Sur chaque poste, connecté avec le compte administrateur local :

1. **Paramètres > Système > À propos > Renommer ce PC (options avancées)**
2. Onglet **Nom de l'ordinateur** > bouton **Modifier**
3. Sélectionner **Domaine**, saisir `tssr.lan`
4. Fournir un compte disposant des droits de jonction au domaine (compte administrateur du domaine)
5. Confirmer le message de bienvenue dans le domaine `tssr.lan`
6. Redémarrer le poste

### 3.1 Vérification post-jonction

```powershell
Get-ComputerInfo | Select-Object CsDomain, CsDomainRole
hostname
```

Résultat attendu : `CsDomain = tssr.lan`, `CsDomainRole = MemberWorkstation`.

---

## 4. Déplacement des objets ordinateur en OU

### 4.1 Constat

Par défaut, un ordinateur rejoignant le domaine est placé dans le conteneur **Computers**, non rattaché à une GPO. Seule l'unité d'organisation `OU=_Ordinateurs,OU=Pharmgreen` reçoit l'application des GPO du Sprint 04.

### 4.2 Déplacement

Sur **SRVWIN01**, dans `dsa.msc` (Utilisateurs et ordinateurs Active Directory) :

1. Localiser l'objet ordinateur (CLIWIN01 puis CLIWIN02) dans le conteneur **Computers**
2. Glisser-déposer (ou couper/coller) l'objet vers `OU=_Ordinateurs,OU=Pharmgreen,DC=tssr,DC=lan`

### 4.3 Vérification

```powershell
Get-ADComputer -Identity CLIWIN01 | Select-Object Name, DistinguishedName
Get-ADComputer -Identity CLIWIN02 | Select-Object Name, DistinguishedName
```

Résultat attendu : `DistinguishedName` contenant `OU=_Ordinateurs,OU=Pharmgreen,DC=tssr,DC=lan` pour les deux machines.

---

## 5. Application forcée des GPO

Sur chaque poste, après déplacement dans la bonne OU, une application forcée a été lancée afin de ne pas attendre le cycle de rafraîchissement automatique (90 minutes par défaut) :

```powershell
gpupdate /force
```

Vérification du détail des GPO appliquées :

```powershell
gpresult /r
```

---

## 6. Consultation et modification de la GPO GPO-DOM-LocalAdmin-01

Dans le cadre du test T05, la configuration existante de la GPO a été consultée (sans modification de fond, la configuration s'est révélée déjà correcte) :

Sur **SRVWIN01**, via `gpmc.msc` :

1. `Group Policy Objects` > sélectionner `GPO-DOM-LocalAdmin-01` > clic droit > **Edit**
2. Naviguer vers `Computer Configuration > Policies > Windows Settings > Security Settings > Restricted Groups`
3. Double-clic sur l'entrée **Administrators**
4. Vérification du contenu de **Members of this group** (`TSSR\Domain Admins`) et de **This group is a member of** (vide)

### 6.1 Génération d'un rapport RSOP pour diagnostic

Sur **CLIWIN01** :

```powershell
mkdir C:\Temp
gpresult /h C:\Temp\gpresult.html
```

Le rapport est ensuite ouvert dans un navigateur pour consultation de la section **Restricted Groups**.

---

## 7. Utilisation des outils RSAT Active Directory (correction AGDLP)

Les cmdlets du module **ActiveDirectory** (déjà installé sur SRVWIN01 depuis le Sprint 03) ont été utilisées pour diagnostiquer puis corriger l'affectation des utilisateurs aux groupes de sécurité :

```powershell
Get-ADOrganizationalUnit -Filter * -SearchBase $baseOU -SearchScope OneLevel
Get-ADGroup -Filter "Name -like 'GG-*'"
Get-ADGroupMember -Identity <groupe>
Add-ADGroupMember -Identity <groupe> -Members <utilisateur>
Get-ADUser -Identity <utilisateur> -Properties MemberOf, PrimaryGroup
```

Le détail des scripts exécutés et de leurs résultats est documenté dans `CONFIGURATION.md`, section 4.

---

## 8. Utilisation de la console DHCP (vérification des baux)

Sur **SRVWIN01**, le module **DhcpServer** (installé avec le rôle DHCP en Sprint 03) a été utilisé pour confirmer côté serveur les baux attribués aux deux postes clients :

```powershell
Get-DhcpServerv4Lease -ScopeId 172.16.10.0 | Select-Object IPAddress, HostName, ClientId, LeaseExpiryTime
