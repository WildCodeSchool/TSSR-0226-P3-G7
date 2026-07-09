# CONFIGURATION - Lignes SIP, Softphones et Test d'appel

## 1. Création des lignes SIP (T04)

### 1.1 Périmètre

22 extensions ont été créées, à raison de 2 utilisateurs réels par département (sur les 11 départements Pharmgreen), en s'appuyant sur les comptes déjà existants dans l'Active Directory (`tssr.lan`).

### 1.2 Plan de numérotation

| Extension | Utilisateur | Département |
|---|---|---|
| 8001 | Valentina Ferrari | Communication |
| 8002 | Lukas Rousseau | Communication |
| 8003 | Nadège Rouiller | Développement Logiciel |
| 8004 | Sacha Sacha-David | Développement Logiciel |
| 8005 | Anaïs Fontaine | Direction Financière |
| 8006 | Eva Reyer | Direction Financière |
| 8007 | Gilles Faure | Direction Générale |
| 8008 | Fanny Garnier | Direction Générale |
| 8009 | Brigitte Lucas | Direction Marketing |
| 8010 | Sylvie Charpentier | Direction Marketing |
| 8011 | Ahmed Khalid | R&D |
| 8012 | Aisha Abdullah | R&D |
| 8013 | Kelly Arnaud | RH |
| 8014 | Lionel Marchand | RH |
| 8015 | Nathalie Nicolas | Service Juridique |
| 8016 | Oscar Perrin | Service Juridique |
| 8017 | Arjun Singh | Services Généraux |
| 8018 | Hélena Ivanova | Services Généraux |
| 8019 | Tina Le Hecourt | Systèmes d'Information |
| 8020 | Hassan Liffite | Systèmes d'Information |
| 8021 | Antoine Lambert | Ventes et Dév. Commercial |
| 8022 | Camille Fournier | Ventes et Dév. Commercial |

**Technologie** : PJSIP (recommandée par FreePBX 16, remplace l'ancien chan_sip)
**Secret SIP** (mot de passe d'authentification) : `Azerty1*` pour l'ensemble des comptes, conforme au cahier de charge du projet.

### 1.3 Méthode de création

**Extension de référence (8019)** créée manuellement via **Applications -> Extensions -> Ajouter un poste -> Poste SIP [chan_pjsip]**, avec :
- Extension Utilisateur : 8019
- Nom affiché : Tina Le Hecourt
- Secret : Azerty1*

**21 extensions restantes** créées via le module **Bulk Handler** (Admin -> Bulk Handler -> Postes -> Import), en s'appuyant sur le format CSV exact obtenu par export de l'extension de référence. Étapes :
1. Export du poste 8019 pour obtenir le gabarit CSV exact (colonnes attendues par FreePBX 16)
2. Construction d'un fichier CSV des 21 lignes restantes, reprenant la structure du poste de référence et substituant uniquement : `extension`, `name`, `description`, `id`, `dial`, `user`, `cid_masquerade`, `devicedata`, `callerid`, `mailbox`, `findmefollow_grplist`, `findmefollow_postdest`, `voicemail_vmpwd`, `voicemail_email`
3. Import via **Bulk Handler -> Import -> Postes**, option **"Replace/Update existing data" = Oui**
4. Validation de l'aperçu (Data Validation) puis confirmation de l'import

**Résultat** : 22/22 lignes créées avec succès (Applications -> Extensions -> 22 lignes au total).

> Cette méthode **Bulk Handler** est directement réutilisable pour déployer les 186 comptes SIP restants (208 utilisateurs Pharmgreen au total), en appliquant le même gabarit CSV.

---

## 2. Installation des softphones (T05)

**Client utilisé** : 3CXPhone (legacy standalone softphone), compatible avec tout PBX SIP tiers.

**Source de téléchargement** :

```
https://downloads.3cx.com/downloads/3CXPhone6.msi
```

### 2.1 Configuration du compte SIP - CLIWIN01

| Champ | Valeur |
|---|---|
| Account name | Tina Le Hecourt |
| Caller ID | Tina Le Hecourt |
| Extension | 8019 |
| ID | 8019 |
| Password | Azerty1* |
| Localisation | "I am in the office - local IP" -> 172.16.10.6 |

### 2.2 Configuration du compte SIP - CLIWIN02

| Champ | Valeur |
|---|---|
| Account name | Valentina Ferrari |
| Caller ID | Valentina Ferrari |
| Extension | 8001 |
| ID | 8001 |
| Password | Azerty1* |
| Localisation | "I am in the office - local IP" -> 172.16.10.6 |

### 2.3 Vérification de l'enregistrement

Statut attendu sur l'écran principal du softphone : **"On Hook" / "Available"** (confirme l'enregistrement SIP réussi auprès d'IPBX01).

---

## 3. Test d'appel VoIP (T06)

**Scénario** : appel de CLIWIN01 (extension 8019) vers CLIWIN02 (extension 8001), via composition du numéro `8001` sur le softphone de CLIWIN01.

**Résultat** : appel abouti avec succès, sonnerie reçue sur CLIWIN02, communication établie entre les deux postes.

Ce test valide l'ensemble de la chaîne fonctionnelle :
- Enregistrement SIP des deux extensions auprès d'IPBX01
- Routage interne du plan de numérotation (`from-internal`)
- Ouverture du flux audio (RTP) entre les deux clients

---

## 4. Points de configuration réseau (Sangoma Smart Firewall)

Pour rappel (voir aussi `INSTALL.md`), le client d'administration CLIWIN01 (172.16.10.12) a été explicitement ajouté comme client de confiance lors de l'assistant post-installation, garantissant un accès permanent à l'interface web d'administration.

