# CONFIGURATION.md : Paramétrage GLPI

> Ce document suppose que GLPI est déjà installé et accessible (voir `INSTALL.md`). Il couvre le paramétrage fonctionnel réalisé pour Pharmgreen : entité, catégories de tickets, utilisateurs, inventaire du parc, et les tests de validation effectués.

---

## 1. Création de l'entité Pharmgreen

Une entité dédiée a été créée afin de cloisonner les données de l'entreprise dans GLPI 

**Chemin GLPI** : `Administration -> Entités -> Ajouter`

| Champ | Valeur |
|---|---|
| Nom | Pharmgreen |
| Entité parente | Entité racine |

---

## 2. Catégories de tickets (ITIL)

**Chemin GLPI** : `Configuration -> Intitulés -> Assistance -> Catégories ITIL`

Une arborescence a été mise en place avec **Pharmgreen** comme catégorie parente et 8 sous-catégories filles :

```
Pharmgreen
├── Réseau
├── Matériel
├── Logiciel
├── Active Directory
├── Messagerie
├── Téléphonie
├── Accès et permissions
└── Autre
```

**Procédure pour chaque catégorie fille** :
1. Cliquer sur le bouton **"+"** (ajouter)
2. Nom : *(voir liste ci-dessus)*
3. Champ **"Comme enfant de"** : sélectionner `Pharmgreen`
4. Cliquer sur **Sauvegarder**

> **Point d'attention** : le champ "Comme enfant de" concerne la hiérarchie des catégories de tickets, à ne pas confondre avec le sélecteur d'entité (en haut à droite de l'écran), qui lui définit dans quelle entité GLPI l'élément est créé.

---

## 3. Gestion des utilisateurs

Trois comptes GLPI ont été créés, en cohérence avec les données réelles du fichier RH Pharmgreen (`ListeRH.csv`) pour respecter l'organigramme réel de l'entreprise.

**Chemin GLPI** : `Administration -> Utilisateurs -> Ajouter utilisateur`

### 3.1 Convention appliquée

| Élément | Règle |
|---|---|
| Identifiant | Première lettre du prénom + `.` + nom (ex : `t.lehecourt`) |
| Email | `prenom.nom@pharmgreen.fr` |
| Mot de passe initial | `Azerty1*` |
| Entité | Pharmgreen |
| Récursif | Non |

### 3.2 Comptes créés

| Identifiant | Nom complet | Email | Profil GLPI | Fonction réelle (RH) |
|---|---|---|---|---|
| `t.lehecourt` | Tina Le Hecourt | tina.lehecourt@pharmgreen.fr | Super-Admin | Directrice informatique |
| `c.francois` | Cyril François | cyril.francois@pharmgreen.fr | Technician | Développeur |
| `b.bonnet` | Béatrice Bonnet | beatrice.bonnet@pharmgreen.fr | Technician | Développeur |

> Le champ **"Entité par défaut"** n'apparaît que lorsqu'un utilisateur a plusieurs entités/profils assignés - il ne s'affiche pas ici puisque chaque compte n'a qu'une seule entité (Pharmgreen).

> Les deux autres membres du département SI (Hassan Liffite, Rosalie Roux : Data scientists) n'ont pas encore de compte GLPI ; création reportée à une itération ultérieure.

### 3.3 Procédure détaillée (exemple avec Tina Le Hecourt)

1. **Administration -> Utilisateurs -> Ajouter utilisateur**
2. Renseigner :
   - Identifiant : `t.lehecourt`
   - Nom de famille : `Le Hecourt`
   - Prénom : `Tina`
   - Mot de passe / Confirmation : `Azerty1*`
   - Courriels : `tina.lehecourt@pharmgreen.fr`
3. Cliquer sur **Ajouter** (première sauvegarde)
4. Sur la fiche utilisateur nouvellement créée, section **Habilitation** :
   - Profil : `Super-Admin`
   - Entité : `Entité racine > Pharmgreen`
   - Récursif : `Non`
5. Cliquer sur **+ Ajouter** pour valider l'habilitation

---

## 4. Convention de modèles matériels Pharmgreen

Dans le cadre de la réorganisation du parc informatique (l'entreprise étant historiquement désorganisée côté IT), une convention de standardisation matérielle a été définie :

| Catégorie | Fabricant | Modèle |
|---|---|---|
| Postes fixes | Dell | OptiPlex 7020 |
| Postes portables | Dell | Latitude 5540 |
| Serveurs physiques | Dell | PowerEdge R650 |
| Machines virtuelles | *(vide)* | *(vide)* |

> Pour les VMs (cas de toutes les machines du Sprint 05), les champs Fabricant/Modèle sont volontairement laissés vides : ils n'ont pas de sens physique réel. Cette convention sera appliquée dès l'ajout de postes physiques réels au parc.

---

## 5. Inventaire du parc (Gestion de parc)

**Chemin GLPI** : `Parc -> Ordinateurs → Ajouter`

> **Point d'attention obligatoire avant toute création** : vérifier que l'entité active (en haut à droite de l'écran) est bien **Pharmgreen**, et non l'Entité racine - sinon les techniciens et utilisateurs Pharmgreen n'apparaîtront pas dans les listes déroulantes (Technicien responsable, Utilisateur). Pour changer d'entité : cliquer sur le sélecteur d'entité en haut à droite -> dérouler l'arborescence -> sélectionner **Pharmgreen**.

### 5.1 Machines enregistrées

| Nom | Statut | Type | Système d'exploitation | Technicien responsable | Commentaire |
|---|---|---|---|---|---|
| **SRVLX01** | En Service | Serveur | Debian 13 / x86_64 | Cyril François | Serveur GLPI: IP 172.16.10.5 |
| **SRVWIN01** | En Service | Serveur | Windows Server 2025 / x86_64 | Cyril François | Contrôleur de domaine (AD DS, DNS, DHCP) - IP 172.16.10.1 |
| **CLIWIN02** | En Service | Ordinateur | Windows 11 / x86_64 | Cyril François | Poste client - joint à tssr.lan |

### 5.2 Procédure type (exemple SRVLX01)

1. **Parc -> Ordinateurs -> Ajouter**
2. Onglet "Ordinateur" :
   - Nom : `SRVLX01`
   - Statut : `En Service` (créer la valeur via le bouton "+" si absente)
   - Type d'ordinateur : `Serveur` (idem)
   - Technicien responsable : sélectionner dans la liste (visible uniquement si l'entité active est Pharmgreen)
   - Fabricant / Modèle : laisser vide (VM)
   - Commentaires : note libre décrivant le rôle et l'IP de la machine
3. Cliquer sur **Ajouter**
4. Une fois la fiche créée, aller dans l'onglet **"Systèmes d'exploitation"** :
   - Nom : `Debian` (créer via "+")
   - Version : `13 (Trixie)` (créer via "+")
   - Architecture : `x86_64` (créer via "+")
5. Sauvegarder l'onglet

---

## 6. Tests de validation

### 6.1 Test d'accès webGUI depuis un poste client

**Objectif** : valider que l'interface GLPI est accessible depuis n'importe quel poste du LAN, pas uniquement depuis la machine ayant servi à l'installation.

**Procédure** :
1. Depuis CLIWIN01, ouvrir un navigateur
2. Accéder à `http://172.16.10.5`
3. S'authentifier avec un compte existant (ex : `t.lehecourt` / `Azerty1*`)

**Résultat obtenu** : Page d'authentification affichée, connexion réussie, tableau de bord visible avec les droits Super-Admin.

> **Point d'attention** : ne pas utiliser un login générique type `admin` - seuls les comptes réellement créés dans GLPI (`glpi`, `t.lehecourt`, `c.francois`, `b.bonnet`, `tech`, `normal`, `post-only`) permettent de s'authentifier.

---

## 7. Dépannage DHCP (rencontré à plusieurs reprises durant le sprint)

### 7.1 Symptôme

Un poste client (CLIWIN01 ou CLIWIN02) obtient une adresse **APIPA** (`169.254.x.x`) au lieu d'une adresse du réseau `172.16.10.0/24`.

### 7.2 Cause la plus fréquente : serveur DHCP non autorisé dans l'AD

Sur SRVWIN01, la console DHCP affiche le message : *"A DHCP server must be authorized in the Active Directory before it can assign IP addresses."*

**Résolution** :

1. Ouvrir la console **DHCP** sur SRVWIN01 (Gestionnaire de serveur -> Outils -> DHCP)
2. Clic droit sur le nœud **IPv4** (celui affichant l'icône flèche rouge)
3. Sélectionner **Authorize**
4. Attendre 10-15 secondes, puis rafraîchir (F5)
5. Si le message *"The specified servers are already present in the directory service"* apparaît lors d'une nouvelle tentative : c'est le signe que le serveur **est déjà autorisé**, il s'agit alors d'un simple problème d'affichage de la console MMC - fermer et rouvrir la console DHCP pour rafraîchir la vue

### 7.3 Vérification complémentaire via PowerShell (sur SRVWIN01)

```powershell
Get-DhcpServerInDC
```

Doit lister `srvwin01.tssr.lan` avec son adresse IP.

Si besoin, redémarrer le service :

```powershell
Restart-Service DHCPServer
```

### 7.4 Sur le poste client concerné

```powershell
ipconfig /release
ipconfig /renew
```

### 7.5 Autres points à vérifier si le problème persiste

- Configuration réseau de la VM cliente dans VirtualBox : Adaptateur activé, **Attaché à : Réseau interne**, **Nom : LAN-Pharmgreen** (identique, casse et espaces compris, à celui de SRVWIN01)
- Câble virtuel bien connecté (case "Virtual Cable Connected" cochée)

---

## 8. Points ouverts / améliorations futures

- Créer les comptes GLPI restants pour Hassan Liffite et Rosalie Roux
- Mettre en place la synchronisation LDAP GLPI ↔ Active Directory
- Ajouter FW01 (pfSense) à l'inventaire du parc
- Appliquer la convention de modèles matériels aux futurs postes physiques ajoutés au parc

