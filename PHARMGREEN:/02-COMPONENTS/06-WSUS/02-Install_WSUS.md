# INSTALLATION WSUS

---

## 1. Préparation de la VM (méthode par clonage)

Contrairement à une création classique, SRVWIN04 a été obtenue en **clonant** une VM Windows Server autonome/template existante (jamais jointe à un domaine), afin de gagner du temps sur l'installation de l'OS.

### 1.1 Vérifications et corrections post-clonage dans VirtualBox

**Adresse MAC** - à régénérer systématiquement après un clonage, pour éviter tout conflit avec la VM d'origine :

1. Sélectionner **SRVWIN04** -> **Configuration** -> **Réseau**
2. Onglet Adaptateur 1 : cliquer sur l'icône d'actualisation à côté du champ **MAC Address**

**Interfaces réseau superflues** - le template clone comportait deux cartes réseau actives (une résiduelle) :

1. Onglet **Adaptateur 2** : décocher **"Activer l'interface réseau"**
2. Onglet **Adaptateur 1** : conserver **Activé**, **Attaché à : Réseau interne**, **Nom : LAN-Pharmgreen**

### 1.2 Diagnostic de l'état hérité du template

Une fois la VM démarrée, en PowerShell (admin) :

```powershell
hostname
Get-NetIPConfiguration
```

> Le template avait hérité d'une IP statique `172.16.10.10` sur une interface, et une seconde interface en APIPA - à corriger avant toute autre configuration.

---

## 2. Configuration réseau définitive

### 2.1 Suppression de l'ancienne IP et attribution de l'IP finale

```powershell
Remove-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 172.16.10.10 -Confirm:$false
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 172.16.10.4 -PrefixLength 24 -DefaultGateway 172.16.10.254
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 172.16.10.1
```

### 2.2 Vérification

```powershell
Get-NetIPConfiguration
```

Résultat attendu :

| Paramètre | Valeur |
|---|---|
| IPv4Address | 172.16.10.4 |
| IPv4DefaultGateway | 172.16.10.254 |
| DNSServer | 172.16.10.1 |

---

## 3. Renommage de la machine

```powershell
Rename-Computer -NewName "SRVWIN04" -Restart
```

Vérification après redémarrage :

```powershell
hostname
```

---

## 4. Jonction au domaine tssr.lan

```powershell
Add-Computer -DomainName "tssr.lan" -Credential (Get-Credential) -Restart
```

Renseigner les identifiants d'un compte administrateur du domaine (`TSSR\Administrateur`) dans la fenêtre d'authentification.

Vérification après redémarrage :

```powershell
Get-ComputerInfo | Select CsDomain, CsDomainRole
```

Résultat attendu :

| Paramètre | Valeur |
|---|---|
| CsDomain | tssr.lan |
| CsDomainRole | MemberServer |

> **Point d'attention** : si la jonction échoue silencieusement (retour à `WORKGROUP` / `StandaloneServer` malgré une exécution sans erreur apparente), vérifier avant de retenter :
> - Résolution DNS du domaine : `Resolve-DnsName tssr.lan`
> - Accessibilité LDAP du contrôleur de domaine : `Test-NetConnection srvwin01.tssr.lan -Port 389`
> - Ressaisir soigneusement les identifiants avec le préfixe `TSSR\` lors de la nouvelle tentative

---

## 5. Installation du rôle WSUS

### 5.1 Créer le dossier de contenu

```powershell
New-Item -Path "C:\WSUS" -ItemType Directory
```

> Adapter le chemin selon les disques réellement disponibles sur la VM (`C:\` uniquement dans notre cas, pas de disque `D:\`).

### 5.2 Installer le rôle

```powershell
Install-WindowsFeature -Name UpdateServices -IncludeManagementTools
```

### 5.3 Post-installation

```powershell
cd "C:\Program Files\Update Services\Tools"
.\wsusutil.exe postinstall CONTENT_DIR=C:\WSUS
```

Résultat attendu : `Post install has successfully completed`.

---

## 6. Correctif TLS 1.2 pour .NET Framework

> Nécessaire sur les serveurs clonés depuis un template ancien : le support TLS 1.2 fort n'est parfois pas activé pour .NET Framework, ce qui peut empêcher WSUS de négocier correctement la connexion sécurisée avec les serveurs Microsoft Update.

### 6.1 Vérifier la présence de la clé

```powershell
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319" -Name "SchUseStrongCrypto" -ErrorAction SilentlyContinue
Get-ItemProperty -Path "HKLM:\SOFTWARE\Wow6432Node\Microsoft\.NETFramework\v4.0.30319" -Name "SchUseStrongCrypto" -ErrorAction SilentlyContinue
```

### 6.2 Appliquer le correctif si la clé est absente

```powershell
New-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319" -Name "SchUseStrongCrypto" -Value 1 -PropertyType DWord -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Wow6432Node\Microsoft\.NETFramework\v4.0.30319" -Name "SchUseStrongCrypto" -Value 1 -PropertyType DWord -Force
```

### 6.3 Redémarrer les services

```powershell
Restart-Service WsusService
iisreset
```

> **Remarque importante** : ce correctif est une bonne pratique générale, mais n'a pas résolu à lui seul le blocage de synchronisation rencontré dans ce sprint - voir `CONFIGURATION.md` pour le diagnostic complet de cet incident, dont la cause s'est révélée externe (côté Microsoft).

---


------------------------------------------------------------------------------------------------------------------



# INSTALLATION — WSUS (SRVWIN04)

## 1. Origine de la VM (Sprint 06)

Contrairement aux autres serveurs du projet, **SRVWIN04 a été créée par clonage** d'un template Windows Server existant (plutôt que par installation complète depuis une ISO), puis reconfigurée aux standards du projet.

| Champ | Valeur |
|---|---|
| Nom | SRVWIN04 |
| Système | Windows Server 2022 *(hérité du template cloné)* |
| Réseau | Adaptateur 1 → Réseau interne → `LAN-Pharmgreen` |

### 1.1 Reconfiguration post-clonage

Après clonage, la VM a été renommée et rejointe au domaine :

```powershell
Rename-Computer -NewName "SRVWIN04" -Restart
```

Jonction au domaine :
```powershell
$cred = Get-Credential TSSR\Administrateur
Add-Computer -DomainName "tssr.lan" -Credential $cred -Restart
```

### 1.2 IP statique

| Paramètre | Valeur |
|---|---|
| Adresse IP | 172.16.10.4 /24 |
| Passerelle | 172.16.10.254 (FW01) |
| DNS | 172.16.10.1 (SRVWIN01) |

### 1.3 Vérification de la jonction au domaine

```powershell
Get-ComputerInfo | Select CsDomain, CsDomainRole
hostname
```

Résultat attendu : `CsDomain = tssr.lan`, `CsDomainRole = MemberServer`, `hostname = SRVWIN04`.

---

## 2. Installation du rôle WSUS

### 2.1 Créer le dossier de stockage du contenu

```powershell
New-Item -Path "C:\WSUS" -ItemType Directory
```

> ℹ️ Le chemin dépend de la configuration disque de la VM — utiliser `C:\WSUS` en l'absence de disque secondaire dédié (`D:\`).

### 2.2 Installer le rôle

```powershell
Install-WindowsFeature -Name UpdateServices -IncludeManagementTools
```

Vérifier le résultat : `Success : True`.

### 2.3 Post-installation — initialiser le dossier de contenu

```powershell
cd "C:\Program Files\Update Services\Tools"
.\wsusutil.exe postinstall CONTENT_DIR=C:\WSUS
```

> ⚠️ Le chemin passé à `CONTENT_DIR` doit correspondre exactement au disque réellement disponible sur la VM — une erreur de lettre de lecteur (ex. `D:\WSUS` sur une machine sans disque D:) provoque l'échec `The device is not ready`.

Cette étape peut prendre plusieurs minutes (création de la base de données interne WSUS/WID).

---

## 3. Premier lancement de la console WSUS

Au premier lancement (**Outils → Windows Server Update Services**), l'assistant de configuration se déclenche :

1. **Before You Begin** → Suivant
2. **Join the Microsoft Update Improvement Program** → décoché → Suivant
3. **Choose Upstream Server** → **Synchronize from Microsoft Update** → Suivant
4. **Specify Proxy Server** → aucun proxy nécessaire (accès direct via NAT FW01) → Suivant
5. **Connect to Upstream Server** → **Start Connecting**
6. **Choose Languages** → Français, Anglais (selon besoin) → Suivant
7. **Choose Products** → sélectionner les produits Windows Server / Windows 11 concernés par le parc Pharmgreen → Suivant
8. **Choose Classifications** → Critical Updates, Security Updates *(a minima)* → Suivant
9. **Set Sync Schedule** → synchronisation manuelle ou planifiée quotidienne → Suivant
10. **Finished** → décocher **"Begin initial synchronization"** si l'on préfère lancer la synchro manuellement après coup

---

## 4. Création des groupes de machines

**Console WSUS → Computers → All Computers → Add Computer Group...**

| Groupe | Usage |
|---|---|
| Serveurs | Machines serveurs Windows Server |
| Postes clients | Postes Windows 11 (CLIWIN01, CLIWIN02) |

Procédure : clic droit sur **All Computers → Add Computer Group... → nom du groupe → Add**.

---

## 5. Approbation automatique des mises à jour

**Console WSUS → Options → Automatic Approvals → Update Rules** :

Règle **Default Automatic Approval Rule** :
- Déclencheur : **Critical Updates, Security Updates**
- Cible : **all computers**

> ℹ️ Cette étape peut être bloquée (`Cannot save configuration because the server is synchronizing`) si une synchronisation est active ou en tentative répétée — WSUS verrouille certains paramètres dans ce cas. Relancer la sauvegarde après l'échec d'une tentative de synchronisation, moment où le verrou se libère temporairement.

---

## 6. Déploiement côté client (une fois la synchronisation opérationnelle)

Sur un poste client déjà membre du domaine (ex. CLIWIN01) :
```cmd
gpupdate /force
wuauclt /detectnow
```

Vérifier dans la console WSUS que le poste apparaît dans **Unassigned Computers**, puis l'assigner manuellement au groupe **Postes clients**.




## 7. Vérification finale de l'installation

- [ ] Rôle WSUS installé et console accessible via Gestionnaire de serveur -> Outils -> Windows Server Update Services
- [ ] SRVWIN04 correctement nommée et jointe au domaine tssr.lan
- [ ] IP statique 172.16.10.4 confirmée
- [ ] Dossier de contenu C:\WSUS créé et référencé dans la post-install

La configuration fonctionnelle (groupes de machines, règles d'approbation) et le suivi de l'incident de synchronisation sont détaillés dans **`CONFIGURATION.md`**.
