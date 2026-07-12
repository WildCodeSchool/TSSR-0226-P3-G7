# Sprint 07 — Documentation de la messagerie Pharmgreen (T13)

**Projet :** Pharmgreen
**Sprint :** SP07 — Déploiement de la messagerie interne
**Domaine mail :** pharmgreen.fr
**Domaine AD :** tssr.lan
**Rédigé par :** Technicien SI (TSSR)
**Date :** Juillet 2026

---

## 1. Contexte et objectifs

Le Sprint 07 avait pour objectif de déployer une solution de messagerie électronique interne pour l'ensemble des 211 employés de Pharmgreen, intégrée à l'annuaire Active Directory existant (domaine `tssr.lan`), et de valider son bon fonctionnement par des tests d'envoi/réception sur des postes clients.

Les tâches couvertes par ce document (T08 à T13) sont :

| Tâche | Description | Statut |
|---|---|---|
| T08 | Installation du serveur de messagerie | ✅ Terminée |
| T09 | Configuration du domaine mail | ✅ Terminée |
| T10 | Création des boîtes mail pour les utilisateurs AD | ✅ Terminée |
| T11 | Configuration des clients mail (CLIWIN01, CLIWIN02) | ✅ Terminée |
| T12 | Test d'envoi et de réception entre 2 clients | ✅ Terminée |
| T13 | Documentation de la messagerie | ✅ Terminée (ce document) |

---

## 2. Infrastructure serveur

### 2.1 Serveur de messagerie

| Paramètre | Valeur |
|---|---|
| Nom de la VM | SRVLX01 |
| Système d'exploitation | Debian |
| Adresse IP | 172.16.10.5 |
| Solution installée | iRedMail 1.8.3 |
| Domaine mail | pharmgreen.fr |
| Compte administrateur | postmaster@pharmgreen.fr |

SRVLX01 héberge également GLPI (Sprint 05), ce qui a nécessité une résolution de conflit de port entre Apache et Nginx (utilisé par iRedMail) :

- **Nginx** conserve le port **80/443** pour iRedMail (webmail, iRedAdmin)
- **Apache** est migré sur le port **8080** (modification de `ports.conf`)
- Une règle **nftables** a été ajoutée pour autoriser le trafic sur le nouveau port

### 2.2 Paramètres de connexion des clients mail

Ces paramètres ont été utilisés pour la configuration de tous les clients Outlook :

| Paramètre | Valeur |
|---|---|
| Serveur IMAP entrant | 172.16.10.5 |
| Port IMAP | 993 (chiffrement SSL/TLS) |
| Serveur SMTP sortant | 172.16.10.5 |
| Port SMTP | 587 (chiffrement STARTTLS) |
| Mot de passe standard | Azerty1* |

---

## 3. T10 — Création des boîtes mail

### 3.1 Méthode

1. **Export des utilisateurs AD** depuis SRVWIN01 (Contrôleur de Domaine) via un script PowerShell, avec comme base de recherche `OU=_Utilisateurs,OU=Pharmgreen,DC=tssr,DC=lan`
2. **Transfert du fichier CSV** vers SRVLX01 via SCP
3. **Génération des comptes mail** via un script Python (`create_mailboxes.py`) qui :
   - Lit le fichier CSV exporté
   - Transforme les noms/prénoms accentués (translittération via le module `unicodedata`) pour respecter la convention `prenom.nom@pharmgreen.fr`
   - Génère les instructions SQL via `create_mail_user_SQL.sh`
   - Exécute les insertions dans la base `vmail.mailbox` (MySQL/MariaDB) via `mysql --defaults-file=/etc/mysql/debian.cnf`

### 3.2 Résultat

- **208 boîtes mail** créées avec succès pour les utilisateurs Pharmgreen
- **+ 1 compte postmaster** (administration)
- **Total : 209 comptes** actifs dans `vmail.mailbox`

### 3.3 Point de sécurité

L'accès root SSH à SRVLX01 avait été temporairement élargi (`PermitRootLogin yes`) pour faciliter le dépannage durant la mise au point du script de création des boîtes mail. Ce paramètre a été **restauré à sa valeur sécurisée** (`PermitRootLogin prohibit-password`) une fois T10 validée.

---

## 4. T11 — Configuration des clients mail

### 4.1 Postes concernés

| Poste | Utilisateur | Adresse mail |
|---|---|---|
| CLIWIN01 | Valentina Ferrari | valentina.ferrari@pharmgreen.fr |
| CLIWIN02 | Lukas Rousseau | lukas.rousseau@pharmgreen.fr |

### 4.2 Choix du client : Outlook classique

Le **nouveau Outlook** n'a pas pu être utilisé : il impose une synchronisation via le cloud Microsoft, incompatible avec ce réseau LAN isolé sans accès Internet. **Outlook classique** a donc été retenu, configuré manuellement en IMAP.

### 4.3 Procédure de configuration (identique sur les deux postes)

1. Lancer **Outlook (classic)**
2. Passer les écrans de licence Office (Ignorer / Accepter / Suivant / *"Ne pas envoyer de données facultatives"*)
3. Saisir l'adresse mail → **Options avancées** → **Configuration avancée** → sélectionner **IMAP**
4. Renseigner manuellement les paramètres serveur (voir tableau §2.2)
5. Saisir le mot de passe → accepter l'avertissement de certificat SSL auto-signé (**Oui**)

### 4.4 Résultat

Les deux postes sont configurés et connectés avec succès.

![Outlook connecté sur CLIWIN02](images/01_outlook_cliwin02_connecte.png)

*Boîte de réception lukas.rousseau@pharmgreen.fr — statut "Connecté", dossiers complets visibles, email de test automatique reçu lors de la configuration (preuve que le circuit IMAP/SMTP fonctionne de bout en bout).*

---

## 5. T12 — Test d'envoi et de réception

### 5.1 Objectif

Valider la messagerie dans les **deux sens** entre les deux postes clients configurés.

### 5.2 Test 1 — Lukas → Valentina

Envoi d'un message "Salutation" depuis lukas.rousseau@pharmgreen.fr vers valentina.ferrari@pharmgreen.fr.

![Test Lukas vers Valentina](images/02_test_lukas_vers_valentina.png)

*Message reçu avec succès dans la boîte de Valentina Ferrari.*

### 5.3 Test 2 — Valentina → Lukas

Envoi d'un message "Salutations distinguées" en retour, depuis valentina.ferrari@pharmgreen.fr vers lukas.rousseau@pharmgreen.fr.

![Test Valentina vers Lukas](images/03_test_valentina_vers_lukas.png)

*Message reçu avec succès dans la boîte de Lukas Rousseau.*

### 5.4 Conclusion T12

✅ Le circuit d'envoi et de réception fonctionne parfaitement **dans les deux sens**, confirmant la bonne intégration IMAP/SMTP entre les clients Outlook et le serveur iRedMail (SRVLX01).

---

## 6. Point complémentaire — Carnet d'adresses local

### 6.1 Constat

Lors des tests, il a été observé que le **Carnet d'adresses** d'Outlook restait vide par défaut, empêchant les utilisateurs de retrouver facilement leurs collègues sans connaître leur adresse mail exacte.

![Carnet d'adresses vide](images/04_carnet_adresses_vide.png)

### 6.2 Cause

iRedMail (backend MySQL/MariaDB dans cette installation) ne fournit pas nativement d'annuaire centralisé consultable depuis le client mail — contrairement à un serveur **Microsoft Exchange**, qui propose une **GAL (Global Address List)** synchronisée automatiquement. Cette fonctionnalité nécessiterait la mise en place d'un serveur **LDAP** dédié et d'une synchronisation MySQL → LDAP, ce qui représente un chantier de plusieurs heures, hors périmètre du sprint actuel.

### 6.3 Solution retenue : import CSV de contacts

Une solution pragmatique a été mise en œuvre : l'**import d'un fichier CSV de contacts** dans le dossier Contacts d'Outlook, sur chaque poste concerné.

**Fichier `contacts.csv` utilisé :**

![Contenu du fichier CSV](images/05_contenu_csv_contacts.png)

**Procédure appliquée (sur CLIWIN01 et CLIWIN02) :**

1. Créer le fichier `contacts.csv` (colonnes : `First Name`, `Last Name`, `E-mail Address`)
2. Dans Outlook : **Fichier → Ouvrir et exporter → Importer/Exporter → Importer à partir d'un autre programme ou fichier**
3. Sélectionner **"Valeurs séparées par une virgule"**
4. Parcourir et sélectionner le fichier `contacts.csv`
5. Choisir le dossier de destination **Contacts (uniquement cet ordinateur)**
6. Valider le mapping des champs (Nom, Prénom, E-mail)
7. **Point de vigilance :** vérifier que l'adresse mail est bien mappée sur le champ **"Adresse de courrier..."** et non sur le champ "Adresses" (adresse postale) — une correction manuelle a été nécessaire sur CLIWIN01 lors du premier import
8. Activer le lien vers le carnet d'adresses : clic droit sur le dossier **Contacts** → **Propriétés** → onglet **"Carnet d'adresses Outlook"** → cocher **"Afficher ce dossier en tant que carnet d'adresses"**

### 6.4 Résultat

**CLIWIN01 :**

![Carnet d'adresses fonctionnel sur CLIWIN01](images/06_carnet_adresses_cliwin01_final.png)

*Les deux contacts (Ferrari, Rousseau) apparaissent avec leurs adresses mail correctes.*

**CLIWIN02 :**

![Contact Ferrari sur CLIWIN02](images/07_contacts_cliwin02_ferrari.png)

![Contact Rousseau sur CLIWIN02](images/08_contacts_cliwin02_rousseau.png)

*Import réussi du premier coup, sans correction nécessaire.*

### 6.5 Limites connues de cette solution

- ⚠️ **Pas de synchronisation automatique** : chaque utilisateur doit importer le fichier CSV individuellement dans son propre Outlook
- ⚠️ **Pas d'annuaire centralisé** de type GAL Exchange
- ⚠️ **Maintenance manuelle** : toute création ou modification de compte impose un ré-import du CSV mis à jour sur chaque poste concerné
- Cette solution a été validée uniquement sur les 2 postes de test (CLIWIN01, CLIWIN02). Pour un déploiement aux 208 comptes réels, une industrialisation serait nécessaire (script de génération CSV complet, déploiement via GPO ou script de connexion)

### 6.6 Piste d'amélioration future (hors périmètre du sprint)

La mise en place d'un serveur **OpenLDAP** synchronisé avec la base MySQL de iRedMail, exposé comme service d'annuaire LDAP dans Outlook, permettrait d'obtenir un véritable carnet d'adresses centralisé et automatiquement à jour. Cette piste est identifiée mais **non réalisée**, compte tenu du temps estimé (plusieurs heures) et de son caractère hors-périmètre des tâches T08–T13.

---

## 7. Synthèse Sprint 07 — Messagerie

| Élément | Statut |
|---|---|
| Serveur iRedMail opérationnel | ✅ |
| 208 boîtes mail créées | ✅ |
| Sécurité SSH restaurée | ✅ |
| Client Outlook configuré (CLIWIN01) | ✅ |
| Client Outlook configuré (CLIWIN02) | ✅ |
| Test envoi/réception bidirectionnel | ✅ |
| Carnet d'adresses local (2 postes test) | ✅ |
| Carnet d'adresses centralisé (LDAP) | ⏳ Piste future, hors périmètre |

Le Sprint 07 — Messagerie est **fonctionnellement validé** : le serveur, les comptes, les clients et les échanges de mails sont opérationnels et testés avec succès.

