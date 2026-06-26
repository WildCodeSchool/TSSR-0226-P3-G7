# GPO - Configuration
>Le GPO est un ensemble de règles et de paramètres applicables à des utilisateurs ou des ordinateurs au sein d'un réseau.
>La Documentation des 7 GPO créées durant le Sprint 04 du projet Pharmgreen.
> Chaque GPO est documentée avec son objectif, son chemin de configuration et ses paramètres.

---

## Prérequis

Avant de créer les GPO, les éléments suivants doivent être en place :

- SRVWIN01 promu contrôleur de domaine (Sprint 03) 
- Structure des OU créée dans l'AD (Sprint 03) 
- CLIWIN02 intégrée au domaine tssr.lan et déplacée dans `OU=_Ordinateurs` 
- Console GPMC accessible via la touche windows + R `gpmc.msc` sur SRVWIN01 

---

## Méthode générale de création d'une GPO

Chaque GPO a été créée selon la procédure suivante :

**Étape 1 - Créer la GPO**
```
GPMC → Group Policy Objects -> clic droit -> New -> saisir le nom → OK
```

**Étape 2 - Éditer la GPO**
```
Clic droit sur la GPO -> Edit -> naviguer vers le chemin indiqué -> configurer les paramètres
```

**Étape 3 - Lier la GPO à la cible**
```
Clic droit sur l'OU ou le domaine cible -> Link an Existing GPO -> sélectionner la GPO -> OK
```

---

## T01 - GPO-DOM-PasswordPolicy-01

### Objectif
Imposer des règles strictes sur les mots de passe de tous les utilisateurs du domaine pour renforcer la sécurité des comptes.

### Cible
Liée à : **tssr.lan** (domaine entier)

### Pourquoi Computer Configuration ?
La politique de mot de passe est une politique de domaine imposée par le contrôleur de domaine. Elle se configure obligatoirement sous `Computer Configuration`; c'est une règle Microsoft indépendante du type d'utilisateur concerné.

### Chemin dans l'éditeur
---
<img width="517" height="257" alt="Chemin_GPO-dans-éditeur" src="https://github.com/user-attachments/assets/0d0eceae-3572-4c76-83e7-3da8859538eb" />

---

### Paramètres configurés

| Paramètre | Valeur | Explication |
|---|---|---|
| Enforce password history | 5 passwords | Empêche la réutilisation des 5 derniers mots de passe |
| Maximum password age | 90 days | Le mot de passe expire tous les 90 jours |
| Minimum password age | 1 day | Empêche de changer plusieurs fois le même jour |
| Minimum password length | 8 characters | Le mot de passe doit faire au moins 8 caractères |
| Password must meet complexity requirements | Enabled | Majuscule + minuscule + chiffre + caractère spécial obligatoires |

### Validation
Cette GPO apparaît dans `gpresult /r` sous **COMPUTER SETTINGS -> Applied Group Policy Objects** sur CLIWIN02.

---

## T02 - GPO-DOM-AccountLockout-01

### Objectif
Verrouiller automatiquement un compte après plusieurs tentatives de connexion échouées pour protéger le domaine contre les attaques par force brute.

### Cible
Liée à : **tssr.lan** (domaine entier)

### Chemin dans l'éditeur
---

<img width="562" height="252" alt="Chemin_GPO_dans-editeur2" src="https://github.com/user-attachments/assets/64291860-0dbe-4d86-876b-28d81ea7553c" />

---

### Paramètres configurés

| Paramètre | Valeur | Explication |
|---|---|---|
| Account lockout threshold | 5 invalid logon attempts | Le compte se verrouille après 5 erreurs |
| Account lockout duration | 30 minutes | Le compte reste verrouillé 30 minutes |
| Reset account lockout counter after | 30 minutes | Le compteur d'erreurs se remet à zéro après 30 minutes |

### Validation
Cette GPO apparaît dans `gpresult /r` sous **COMPUTER SETTINGS -> Applied Group Policy Objects** sur CLIWIN02.

---

## T03 - GPO-DOM-ControlPanel-Block-01

### Objectif
Empêcher les utilisateurs Pharmgreen d'accéder au panneau de configuration et aux paramètres Windows pour éviter toute modification non autorisée de la configuration des postes.

### Cible
Liée à : **OU=_Utilisateurs, OU=Pharmgreen**

### Pourquoi User Configuration ?
Cette restriction suit **l'utilisateur** - quel que soit le poste sur lequel il se connecte, il ne peut pas accéder au panneau de configuration. Si on le configurait sous `Computer Configuration`, tous les utilisateurs du poste seraient bloqués, y compris les administrateurs.

### Chemin dans l'éditeur

---
<img width="577" height="185" alt="Chemin_GPO_dans-editeur3" src="https://github.com/user-attachments/assets/89bbce29-d3a2-4b82-af88-20515b969106" />

---

### Paramètres configurés

| Paramètre | Valeur | Explication |
|---|---|---|
| Prohibit access to Control Panel and PC Settings | Enabled | Bloque l'accès au panneau de configuration et aux paramètres Windows |

### Validation
Cette GPO apparaît dans `gpresult /r` sous **USER SETTINGS -> Applied Group Policy Objects** lors de la connexion avec le compte domaine `TSSR\valentina.ferrari` sur CLIWIN02. 

---

## T04 - GPO-DOM-LocalAdmin-01

### Objectif
Garantir que seul le groupe `Domain Admins` dispose des droits d'administrateur local sur les postes clients, empêchant ainsi les utilisateurs standards d'avoir des privilèges d'administration sur leur machine.

### Cible
Liée à : **OU=_Ordinateurs, OU=Pharmgreen**

### Pourquoi Computer Configuration ?
Cette restriction s'applique à la **machine** - elle définit qui est administrateur local sur le poste, indépendamment de l'utilisateur connecté.

### Chemin dans l'éditeur
---
<img width="560" height="205" alt="Chemin_GPO-dans-editeur4" src="https://github.com/user-attachments/assets/e60b6dfc-97a7-4ed9-9c47-2bb21179d680" />

---

### Paramètres configurés

| Paramètre | Valeur | Explication |
|---|---|---|
| Groupe | Administrators | Groupe local des administrateurs de la machine |
| Members of this group | TSSR\Domain Admins | Seul Domain Admins est administrateur local |

### Validation
Cette GPO apparaît dans `gpresult /r` sous **COMPUTER SETTINGS -> Applied Group Policy Objects** sur CLIWIN02. 

---

## T05 - GPO-DOM-PowerShell-Security-01

### Objectif
Sécuriser l'utilisation de PowerShell sur les postes clients en n'autorisant que les scripts signés numériquement et en activant la journalisation des scripts exécutés.

### Cible
Liée à : **OU=_Ordinateurs, OU=Pharmgreen**

### Chemin dans l'éditeur
---
<img width="572" height="215" alt="Chemin_GPO-dans-editeur5" src="https://github.com/user-attachments/assets/650912ac-9e58-4082-a990-39bdcfaf49de" />

---

### Paramètres configurés

| Paramètre | Valeur | Explication |
|---|---|---|
| Turn on Script Execution | Enabled -> Allow only signed scripts | Seuls les scripts signés numériquement peuvent s'exécuter |
| Turn on PowerShell Script Block Logging | Enabled | Toutes les commandes PowerShell sont enregistrées dans les journaux Windows |

### Validation
Cette GPO apparaît dans `gpresult /r` sous **COMPUTER SETTINGS -> Applied Group Policy Objects** sur CLIWIN02. 

---

## T06 - GPO-DOM-Custom-01-Fond d'écran Pharmgreen

### Objectif
Appliquer un fond d'écran commun à tous les utilisateurs Pharmgreen pour renforcer l'identité visuelle de l'entreprise et uniformiser l'apparence des postes.

### Cible
Liée à : **OU=_Utilisateurs, OU=Pharmgreen**

### Pourquoi User Configuration ?
Le fond d'écran suit **l'utilisateur** ; il s'applique quel que soit le poste sur lequel l'utilisateur se connecte.

### Chemin dans l'éditeur

---

<img width="580" height="202" alt="Chemin_GPO-dans-editeur6" src="https://github.com/user-attachments/assets/e16d310d-4d4f-4fa7-bf4b-e85e3e5f683a" />

---

### Paramètres configurés

| Paramètre | Valeur | Explication |
|---|---|---|
| Desktop Wallpaper | Enabled | Active la politique de fond d'écran |
| Wallpaper Name | C:\Windows\Web\Wallpaper\Windows\img0.jpg | Chemin du fond d'écran |
| Wallpaper Style | Fill | Le fond d'écran remplit tout l'écran |

### Validation

Cette GPO apparaît dans `gpresult /r` sous **USER SETTINGS -> Applied Group Policy Objects** lors de la connexion avec le compte domaine `TSSR\valentina.ferrari` sur CLIWIN02.

---

## T07 - GPO-DOM-Custom-02-Désactiver clé USB

### Objectif

Bloquer l'accès à tous les périphériques de stockage amovibles (clés USB, disques externes) sur les postes clients pour prévenir les fuites de données et les infections par des supports externes.

### Cible

Liée à : **OU=_Ordinateurs, OU=Pharmgreen**

### Pourquoi Computer Configuration ?
Le blocage s'applique à la **machine** - aucun utilisateur, quel qu'il soit, ne peut utiliser de périphérique de stockage amovible sur ce poste.

### Chemin dans l'éditeur


---

<img width="580" height="210" alt="Chemin_GPO_dans-editeur7" src="https://github.com/user-attachments/assets/9a53c81a-2ac1-409b-a281-53e5b1536f8b" />

---

### Paramètres configurés

| Paramètre | Valeur | Explication |
|---|---|---|
| All Removable Storage classes: Deny all access | Enabled | Bloque l'accès à tous les périphériques de stockage amovibles |

### Validation
Cette GPO apparaît dans `gpresult /r` sous **COMPUTER SETTINGS -> Applied Group Policy Objects** sur CLIWIN02. 

---

## T08 - Tests de validation sur CLIWIN02

### Procédure de test

**Étape 1 - Déplacer CLIWIN02 dans la bonne OU**

Sur SRVWIN01 dans Active Directory Users and Computers :

```
tssr.lan -> Computers -> clic droit sur CLIWIN02 -> Move
-> Pharmgreen -> _Ordinateurs -> OK

```

**Étape 2 - Forcer l'application des GPO sur CLIWIN02**

```powershell
gpupdate /force
```
Résultat attendu :
```
Computer Policy update has completed successfully.
User Policy update has completed successfully.
```

**Étape 3 - Vérifier les GPO appliquées**

```powershell
gpresult /r
```

### Résultats obtenus

**COMPUTER SETTINGS - connecté avec wilder (compte local)**

| GPO | Statut |
|---|---|
| Default Domain Policy | Appliquée |
| GPO-DOM-PasswordPolicy-01 | Appliquée |
| GPO-DOM-AccountLockout-01 | Appliquée |
| GPO-DOM-LocalAdmin-01 | Appliquée |
| GPO-DOM-PowerShell-Security-01 | Appliquée |
| GPO-DOM-Custom-02-Désactiver clé USB | Appliquée |

**USER SETTINGS - connecté avec TSSR\valentina.ferrari (compte domaine)**

| GPO | Statut |
|---|---|
| GPO-DOM-Custom-01-Fond d'écran Pharmgreen | Appliquée |
| GPO-DOM-ControlPanel-Block-01 | Appliquée |

### Pourquoi deux connexions différentes ?

Les GPO de type `User Configuration` ne s'appliquent qu'aux **comptes du domaine**. Le compte `wilder` étant un compte local, il ne reçoit pas les GPO utilisateur. La connexion avec `TSSR\valentina.ferrari` (compte AD importé depuis le CSV) a permis de valider les GPO utilisateur.

---

## Résumé final

| Tâche | GPO | Type | Cible | Statut |
|---|---|---|---|---|
| T01 | GPO-DOM-PasswordPolicy-01 | Computer | tssr.lan | ok |
| T02 | GPO-DOM-AccountLockout-01 | Computer | tssr.lan | ok |
| T03 | GPO-DOM-ControlPanel-Block-01 | User | OU=_Utilisateurs | ok |
| T04 | GPO-DOM-LocalAdmin-01 | Computer | OU=_Ordinateurs | ok |
| T05 | GPO-DOM-PowerShell-Security-01 | Computer | OU=_Ordinateurs | ok |
| T06 | GPO-DOM-Custom-01-Fond d'écran | User | OU=_Utilisateurs | ok |
| T07 | GPO-DOM-Custom-02-Désactiver clé USB | Computer | OU=_Ordinateurs | ok |
| T08 | Tests sur CLIWIN02 | - | CLIWIN02 | ok |



