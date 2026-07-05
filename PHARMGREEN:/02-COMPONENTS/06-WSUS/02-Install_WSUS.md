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

## 7. Vérification finale de l'installation

- [ ] Rôle WSUS installé et console accessible via Gestionnaire de serveur -> Outils -> Windows Server Update Services
- [ ] SRVWIN04 correctement nommée et jointe au domaine tssr.lan
- [ ] IP statique 172.16.10.4 confirmée
- [ ] Dossier de contenu C:\WSUS créé et référencé dans la post-install

La configuration fonctionnelle (groupes de machines, règles d'approbation) et le suivi de l'incident de synchronisation sont détaillés dans **`CONFIGURATION.md`**.
