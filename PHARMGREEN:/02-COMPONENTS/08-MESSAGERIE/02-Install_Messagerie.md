# INSTALLATION: Serveur de messagerie (iRedMail)

## 1. Choix de la solution

Deux solutions de messagerie open source ont été évaluées : **Zimbra** et **iRedMail**. Zimbra ne proposant pas de support officiel pour Debian, **iRedMail 1.8.3** a été retenu.

## 2. Prérequis serveur

| Paramètre | Valeur |
|---|---|
| VM cible | SRVLX01 |
| Système d'exploitation | Debian |
| Adresse IP | 172.16.10.5 |
| Service déjà présent | GLPI (Apache, PHP 8.3, MariaDB) |

SRVLX01 héberge déjà GLPI depuis le Sprint 05 : l'installation de la messagerie doit donc cohabiter avec les services existants sans les perturber.

## 3. Téléchargement et préparation du script d'installation

```bash
wget https://github.com/iredmail/iRedMail/archive/refs/tags/1.8.3.tar.gz
tar xzf 1.8.3.tar.gz
cd iRedMail-1.8.3
chmod +x iRedMail.sh
```

## 4. Lancement de l'assistant d'installation

```bash
bash iRedMail.sh
```

L'assistant interactif propose les choix suivants :

1. **Répertoire de stockage des boîtes mail** -> valeur par défaut conservée (`/var/vmail`)
2. **Backend d'authentification** -> **MySQL/MariaDB**
3. **Nom de domaine mail** -> `pharmgreen.fr`
4. **Mot de passe du compte postmaster** -> `Azerty1*`
5. **Composants additionnels** à installer -> conservés par défaut : Roundcube (webmail), Amavisd (antivirus/antispam), iRedAdmin (interface d'administration), Fail2ban (protection brute-force)
6. **Confirmation finale** -> validation et lancement de l'installation des paquets

## 5. Configuration réseau (cohabitation avec GLPI)

iRedMail installe **Nginx** pour servir le webmail et iRedAdmin, alors qu'**Apache** (GLPI) est déjà en écoute sur les ports 80/443. Répartition retenue :

| Service | Port |
|---|---|
| Nginx (iRedMail - webmail, iRedAdmin) | 80 / 443 |
| Apache (GLPI) | 8080 |

**Modification apportée** dans `/etc/apache2/ports.conf` et le VirtualHost GLPI, pour faire écouter Apache sur le port 8080. Une règle **nftables** a été ajoutée pour autoriser le trafic entrant sur ce nouveau port.

```bash
systemctl restart apache2
systemctl restart nginx
```

## 6. Vérification post-installation

- Bases de données créées : `vmail`, `amavisd`, `iredadmin`, `iredapd`, `roundcambemail`, `fail2ban`
- Compte `postmaster@pharmgreen.fr` actif et fonctionnel
- Interface **iRedAdmin** accessible : `https://172.16.10.5/iredadmin/`
- **GLPI** toujours accessible sur `http://172.16.10.5:8080/glpi/`

---

## 7. Création des boîtes mail

### 7.1 Méthode

1. **Export des utilisateurs AD** depuis SRVWIN01 (Contrôleur de Domaine) via un script PowerShell, avec comme base de recherche `OU=_Utilisateurs,OU=Pharmgreen,DC=tssr,DC=lan`
2. **Transfert du fichier CSV** vers SRVLX01 via SCP
3. **Génération des comptes mail** via un script Python (`create_mailboxes.py`) qui :
   - Lit le fichier CSV exporté
   - Transforme les noms/prénoms accentués (translittération via le module `unicodedata`) pour respecter la convention `prenom.nom@pharmgreen.fr`
   - Génère les instructions SQL via `create_mail_user_SQL.sh`
   - Exécute les insertions dans la base `vmail.mailbox` (MySQL/MariaDB) via `mysql --defaults-file=/etc/mysql/debian.cnf`

```bash
python3 create_mailboxes.py
```

### 7.2 Résultat

- **208 boîtes mail** créées avec succès pour les utilisateurs Pharmgreen
- **+ 1 compte postmaster** (administration)
- **Total : 209 comptes** actifs dans `vmail.mailbox`

### 7.3 Vérification

```bash
mysql --defaults-file=/etc/mysql/debian.cnf -e "SELECT COUNT(*) FROM vmail.mailbox;"
```

---

Voir [`README.md`](./README.md) pour la vue d'ensemble du composant et [`CONFIGURATION.md`](./CONFIGURATION.md) pour la configuration des clients mail.
