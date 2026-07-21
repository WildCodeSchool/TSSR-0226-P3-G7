# Tests fonctionnels & modification AGDLP

---

## 1. Test T05 : Application des 7 GPO

### 1.1 GPO confirmées fonctionnelles

| GPO | Test réalisé | Résultat |
|---|---|---|
| GPO-DOM-PasswordPolicy-01 | Confirmée appliquée via `gpresult /r` (Computer Settings) | Ok |
| GPO-DOM-AccountLockout-01 | Confirmée appliquée via `gpresult /r` (Computer Settings) | Ok |
| GPO-DOM-ControlPanel-Block-01 | Tentative d'accès au Panneau de configuration / `ms-settings:` refusée | Ok |
| GPO-DOM-Custom-01 (fond d'écran) | Fond d'écran Pharmgreen affiché correctement sur CLIWIN01/02 | Ok |
| GPO-DOM-PowerShell-Security-01 | Politique d'exécution confirmée `AllSigned`, tentative de contournement refusée (`SecurityException`) | Ok |

### 1.2 GPO-DOM-LocalAdmin-01 : Anomalie non résolue

**Configuration vérifiée (correcte) :**
- GPO éditée via `gpmc.msc` > Computer Configuration > Security Settings > Restricted Groups
- Groupe `Administrators`, section **Members of this group** : `TSSR\Domain Admins`
- Section **This group is a member of** : vide (conforme)
- `gpresult /h` confirme cette GPO comme **Winning GPO** pour le paramètre Restricted Groups sur CLIWIN01

**Test réalisé :**
```powershell
Get-LocalGroupMember -Group "Administrators"
```

**Résultat obtenu (non conforme) :**
```
ObjectClass Name                     PrincipalSource
----------- ----                     ---------------
User        CLIWIN01\Administrator   Local
Group        TSSR\Domain Admins      ActiveDirectory
```

Le compte administrateur local intégré n'est pas retiré du groupe, alors que la stratégie calculée (RSOP) ne mentionne que `TSSR\Domain Admins` comme membre attendu.

**Actions de dépannage menées, sans succès :**
1. `gpupdate /force` répété
2. Redémarrage complet de CLIWIN01
3. Suppression du cache de version de la CSE Security (`HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Group Policy\State\Machine\Extension-List\{827D319E-6EAC-11D2-A4EA-00C04F79F83A}`) suivie d'un nouveau `gpupdate /force`

**Éléments confirmés côté journalisation :**
- `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational"` : aucune erreur ni avertissement relatif à Restricted Groups
- `Get-WinEvent -LogName System` (Provider `Microsoft-Windows-GroupPolicy`, ID 1502) : traitement de la stratégie confirmé à plusieurs reprises, succès signalé côté journal système
- `gpresult /h` (rapport HTML RSOP) : section Restricted Groups affiche bien `Administrators → Members: TSSR\Domain Admins`, `Winning GPO: GPO-DOM-LocalAdmin-01`

**Statut** : anomalie non résolue à ce stade. Le paramètre est correctement calculé par le moteur de stratégie de groupe (RSOP) mais ne semble pas appliqué de façon persistante à la base de sécurité locale (SAM). Point mis en pause pour ne pas bloquer le sprint. Pistes de reprise identifiées : comparaison des résultats entre `Get-LocalGroupMember` et `net localgroup Administrators` (écart possible de cache/provider), export direct de la base SAM via `secedit /export /cfg ... /areas GROUP_MGMT`.

### 1.3 GPO-DOM-Custom-02 (USB) : Non testée

Test fonctionnel non réalisable dans l'environnement de lab actuel : aucune clé USB physique disponible pour un passthrough vers les VM CLIWIN01/CLIWIN02 via VirtualBox (Périphériques > USB). La GPO est déployée et confirmée dans les GPO appliquées (`gpresult /r`), mais son comportement réel (blocage effectif du périphérique) n'a pas pu être vérifié.

---

## 2. Test T06 : Connexion utilisateur AD

| Poste | Compte | `whoami` | Niveau d'intégrité | Résultat |
|---|---|---|---|---|
| CLIWIN01 | valentina.ferrari | `tssr\valentina.ferrari` | Medium Mandatory Level | Ok |
| CLIWIN02 | pierre.david | `tssr\pierre.david` | Medium Mandatory Level | Ok |

Connexion réussie sur les deux postes avec un compte de domaine standard (non privilégié). Création automatique du profil utilisateur confirmée. Aucune élévation de privilège constatée (comportement attendu pour un compte standard).

Vérification complémentaire côté AD pour `valentina.ferrari` :
```powershell
Get-ADUser -Identity valentina.ferrari -Properties MemberOf, PrimaryGroup | Select-Object Name, PrimaryGroup, MemberOf
```
`PrimaryGroup` confirmé : `CN=Domain Users,CN=Users,DC=tssr,DC=lan` (comportement standard AD). Cette vérification a mis en évidence l'absence d'appartenance aux groupes AGDLP, traitée en section 4 ci-dessous.

---

## 3. Test T07 : DHCP, DNS et résolution de noms

### 3.1 Attribution DHCP

| Poste | IP obtenue | Serveur DHCP | Bail |
|---|---|---|---|
| CLIWIN01 | 172.16.10.12 | 172.16.10.1 | Confirmé, 8 jours |
| CLIWIN02 | 172.16.10.11 | 172.16.10.1 | Confirmé, 8 jours |

Confirmation croisée côté serveur (SRVWIN01) :
```powershell
Get-DhcpServerv4Lease -ScopeId 172.16.10.0 | Select-Object IPAddress, HostName, ClientId, LeaseExpiryTime
```
```
IPAddress     HostName            ClientId            LeaseExpiryTime
---------     --------            --------            ---------------
172.16.10.11  CLIWIN02.tssr.lan   08-00-27-bc-5d-e7    28/07/2026 15:56:23
172.16.10.12  CLIWIN01.tssr.lan   08-00-27-36-f5-44    28/07/2026 15:54:23
```

### 3.2 Résolution DNS interne

Testée sur les deux postes via `Resolve-DnsName` :

| Nom résolu | Résultat |
|---|---|
| srvwin01.tssr.lan | 172.16.10.1  |
| srvlx01.tssr.lan | 172.16.10.5  |

### 3.3 Résolution DNS externe

Testée via `Resolve-DnsName google.com` sur les deux postes : résolution réussie (A et AAAA), confirmant le bon fonctionnement des forwarders DNS configurés sur SRVWIN01 (8.8.8.8 / 1.1.1.1).

---

## 4. Correction du modèle AGDLP (constat en cours de T06)

### 4.1 Constat initial

Lors de la vérification des groupes de `valentina.ferrari` (§2), l'attribut `MemberOf` est revenu vide. Vérification étendue à l'ensemble des groupes de sécurité créés en Sprint 03 :

```powershell
Get-ADGroup -Filter {GroupCategory -eq "Security" -and Name -notlike "Domain*" -and Name -notlike "*Admins"} | Select-Object Name, @{N="MemberCount";E={(Get-ADGroupMember $_).Count}}
```

Résultat : l'ensemble des 11 groupes globaux (`GG-*`) et des 6 groupes locaux de domaine (`GDL-*`) affichaient un `MemberCount` de 0. Les groupes existaient dans l'AD (créés en Sprint 03) mais n'avaient jamais été peuplés : le placement des utilisateurs en OU par département avait bien été réalisé lors de l'import, mais l'affectation aux groupes de sécurité correspondants n'avait pas été effectuée.

### 4.2 Affectation des utilisateurs à leur groupe global (GG)

Le script suivant affecte chaque utilisateur à son GG de département, en se basant sur son OU actuelle (source fiable, déjà validée en Sprint 03) plutôt que sur une nouvelle lecture du fichier CSV source :

```powershell
$baseOU = "OU=_Utilisateurs,OU=Pharmgreen,DC=tssr,DC=lan"
$deptOUs = Get-ADOrganizationalUnit -Filter * -SearchBase $baseOU -SearchScope OneLevel

$groupNameMapping = @{
    "Ventes-et-Dev-Commercial" = "GG-Ventes-Dev-Commercial"
}

$totalAdded = 0

foreach ($ou in $deptOUs) {
    $deptName = $ou.Name
    $groupName = if ($groupNameMapping.ContainsKey($deptName)) { $groupNameMapping[$deptName] } else { "GG-$deptName" }

    $group = Get-ADGroup -Filter "Name -eq '$groupName'" -ErrorAction SilentlyContinue
    if (-not $group) {
        Write-Host "[ERREUR] Groupe '$groupName' introuvable pour l'OU '$deptName' — ignoré" -ForegroundColor Red
        continue
    }

    $users = Get-ADUser -Filter * -SearchBase $ou.DistinguishedName
    foreach ($user in $users) {
        Add-ADGroupMember -Identity $group -Members $user -ErrorAction SilentlyContinue
        $totalAdded++
    }
    Write-Host "[FAIT] $($users.Count) utilisateur(s) ajouté(s) à '$groupName'" -ForegroundColor Green
}
```

**Point particulier** : l'OU `Ventes-et-Dev-Commercial` ne correspond pas exactement au nom du groupe `GG-Ventes-Dev-Commercial` (absence du second "et"). Une table de correspondance a été utilisée pour ce cas isolé, sur le même principe que la table de correspondance utilisée pour l'import CSV en Sprint 03.

**Résultat : effectifs par groupe (208 utilisateurs au total) :**

| Groupe | Effectif |
|---|---|
| GG-Communication | 21 |
| GG-Developpement-Logiciel | 21 |
| GG-Direction-Financiere | 13 |
| GG-Direction-Generale | 12 |
| GG-Direction-Marketing | 21 |
| GG-R-et-D | 27 |
| GG-RH | 25 |
| GG-Service-Juridique | 8 |
| GG-Services-Generaux | 11 |
| GG-Systemes-Information | 4 |
| GG-Ventes-Dev-Commercial | 45 |

Vérification effectuée indépendamment du script, directement depuis l'état de l'AD :
```powershell
Get-ADGroup -Filter "Name -like 'GG-*'" | Select-Object Name, @{N="MemberCount";E={(Get-ADGroupMember $_).Count}}
```
Effectifs confirmés identiques.

### 4.3 Imbrication des GG dans les GDL de partage

**Décision d'architecture** : tous les départements disposent d'un accès en lecture à un espace documentaire commun. Chaque département dispose également d'un accès en écriture, mais limité à son propre espace dédié ; le cloisonnement réel sera assuré via les permissions **NTFS** appliquées aux sous-dossiers par département (Sprint 09), et non par la création de groupes GDL supplémentaires. Les deux groupes `GDL-Partages-Lecture` et `GDL-Partages-Ecriture` déjà créés en Sprint 03 contiennent donc, à ce stade, les mêmes 11 GG ,le contrôle d'accès fin se fera au niveau des ACL, pas au niveau du groupe.

```powershell
$ggGroups = Get-ADGroup -Filter "Name -like 'GG-*'"
$gdlLecture = Get-ADGroup -Filter "Name -eq 'GDL-Partages-Lecture'"
$gdlEcriture = Get-ADGroup -Filter "Name -eq 'GDL-Partages-Ecriture'"

foreach ($gg in $ggGroups) {
    Add-ADGroupMember -Identity $gdlLecture -Members $gg -ErrorAction SilentlyContinue
    Add-ADGroupMember -Identity $gdlEcriture -Members $gg -ErrorAction SilentlyContinue
}
```

Vérification :
```powershell
Get-ADGroupMember -Identity $gdlLecture | Select-Object Name
Get-ADGroupMember -Identity $gdlEcriture | Select-Object Name
```
Les 11 `GG-*` sont confirmés membres des deux GDL.

### 4.4 Renvoi vers le Sprint 09

Le cloisonnement effectif des droits d'écriture (un département ne peut écrire que dans son propre sous-dossier) sera implémenté au Sprint 09, lors de la création de l'arborescence des dossiers partagés et de la configuration des ACL NTFS correspondantes, ainsi que du mappage du lecteur `I:` via GPO.

---

