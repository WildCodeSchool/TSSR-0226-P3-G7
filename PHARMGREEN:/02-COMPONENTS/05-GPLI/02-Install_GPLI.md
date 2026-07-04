# Installation GLPI

---

## 1. Prérequis

- VM **SRVLX01** créée sous VirtualBox, Debian 13 (Trixie) - CLI
- IP statique configurée : `172.16.10.5 / 24`, passerelle `172.16.10.254`
- Carte réseau attachée au **Réseau interne LAN-Pharmgreen**
- Accès root ou sudo sur SRVLX01

---

## 2. Installation de la stack LAMP

### 2.1 Mise à jour du système et installation d'Apache/MariaDB

```bash
apt update && apt upgrade -y
apt install apache2 mariadb-server -y
```

### 2.2 Installation de PHP 8.3 (dépôt externe sury.org)

> **Point d'attention technique** : PHP 8.4, présent par défaut dans les dépôts Debian 13, n'est **pas compatible** avec GLPI 10.0.x. Il a fallu ajouter le dépôt tiers `sury.org` pour installer PHP 8.3.

```bash
apt install apt-transport-https lsb-release ca-certificates curl -y
curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg
echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" \
  > /etc/apt/sources.list.d/php.list
apt update
apt install php8.3 php8.3-{mysqli,curl,gd,intl,mbstring,xml,zip,bz2,apcu,ldap} -y
a2dismod php8.4
a2enmod php8.3
systemctl restart apache2
```

### 2.3 Sécurisation de MariaDB

```bash
mysql_secure_installation
```

Répondre "Oui" à toutes les questions (définir un mot de passe root, supprimer les utilisateurs anonymes, interdire la connexion root à distance, supprimer la base de test).

### 2.4 Création de la base de données GLPI

```bash
mysql -u root -p
```

```sql
CREATE DATABASE glpi;
CREATE USER 'glpi'@'localhost' IDENTIFIED BY 'Azerty1*';
GRANT ALL PRIVILEGES ON glpi.* TO 'glpi'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 3. Installation de GLPI

### 3.1 Téléchargement et extraction

```bash
cd /var/www/html
wget https://github.com/glpi-project/glpi/releases/download/10.0.17/glpi-10.0.17.tgz
tar -xzf glpi-10.0.17.tgz
chown -R www-data:www-data /var/www/html/glpi
```

### 3.2 Configuration du VirtualHost Apache (`glpi.conf`)

Créer le fichier `/etc/apache2/sites-available/glpi.conf` :

```apache
<VirtualHost *:80>
    ServerName 172.16.10.5
    DocumentRoot /var/www/html/glpi/public

    <Directory /var/www/html/glpi/public>
        AllowOverride All
        Require all granted
        RewriteEngine On
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
    </Directory>
</VirtualHost>
```

Activer le site et le module de réécriture :

```bash
a2ensite glpi.conf
a2enmod rewrite
a2dissite 000-default.conf
systemctl reload apache2
```

### 3.3 Incidents rencontrés et résolus lors de l'installation

| Problème | Cause | Solution |
|---|---|---|
| Erreur DocumentRoot | Chemin `/Var/www/...` avec majuscule | Correction de la casse exacte dans `glpi.conf` |
| Boucle de redirection vers `/install/install.php` | Fichier `.htaccess` manquant dans `public/` | Copie du `.htaccess` fourni par GLPI dans le dossier `public/` |
| Page blanche / erreur d'affichage | Cache navigateur (CLIWIN02) | Vidage du cache (Ctrl+Maj+Suppr) |

### 3.4 Installation via l'interface web

1. Depuis un poste client (CLIWIN02), ouvrir un navigateur
2. Accéder à `http://172.16.10.5`
3. Suivre l'assistant d'installation GLPI en français
4. Choisir **"Installation"** (pas mise à jour)
5. Se connecter à la base existante :
   - Serveur : `localhost`
   - Utilisateur : `glpi`
   - Mot de passe : `Azerty1*`
   - Base de données : `glpi`

### 3.5 Sécurisation post-installation (obligatoire)

Supprimer le dossier d'installation, sous peine de faille de sécurité :

```bash
rm -rf /var/www/html/glpi/install/install.php
```

Activer le cookie de session sécurisé dans `/etc/php/8.3/apache2/php.ini` :

```ini
session.cookie_httponly = 1
```

Redémarrer Apache :

```bash
systemctl restart apache2
```

Changer les mots de passe par défaut des comptes `glpi`, `tech`, `normal`, `post-only` (nouveau mot de passe : `Azerty1*`) via **Administration → Utilisateurs**.

---

## 4. Déploiement des postes clients de test

### 4.1 CLIWIN01 - Création de la VM (VirtualBox)

| Paramètre | Valeur |
|---|---|
| Nom | CLIWIN01 |
| Type / Version | Microsoft Windows / Windows 11 (64-bit) |
| RAM | 4096 Mo |
| vCPU | 2 |
| Disque | 60 Go (VDI, alloué dynamiquement) |
| Réseau | Réseau interne — LAN-Pharmgreen |

> **Incident rencontré** : une erreur `E_INVALIDARG` est survenue lors de la création initiale, un fichier `CLIWIN01.vdi` fantôme existant déjà sur le disque suite à une tentative précédente. Résolution : Gestionnaire de médias virtuels VirtualBox (`Ctrl+D`) → sélection du disque fantôme → Supprimer (suppression physique du fichier) → recréation de la VM.

### 4.2 Installation de Windows 11

1. Démarrer la VM sur l'ISO Windows 11
2. À l'écran de connexion réseau obligatoire, appuyer sur `Shift+F10` pour ouvrir une invite de commandes
3. Taper : `oobe\bypassnro` (contournement du compte Microsoft obligatoire, redémarre automatiquement)
4. Reprendre l'installation, créer un compte local :
   - Nom : `wilder`
   - Mot de passe : `Azerty1*`
5. Terminer l'installation (paramètres de confidentialité par défaut)

### 4.3 Renommage de la machine

Une fois sur le bureau, ouvrir PowerShell en administrateur :

```powershell
Rename-Computer -NewName "CLIWIN01" -Restart
```

### 4.4 Jonction au domaine tssr.lan

1. Ouvrir la fenêtre "Exécuter" (`Windows + R`)
2. Taper `sysdm.cpl` puis Entrée
3. Onglet "Nom de l'ordinateur" → cliquer sur **Modifier...**
4. Sélectionner **Domaine**, saisir : `tssr.lan`
5. Renseigner les identifiants d'un administrateur du domaine (`TSSR\Administrateur`)
6. Redémarrer la VM une fois la jonction confirmée

### 4.5 Vérification de la connectivité réseau

```powershell
ipconfig /all
```

Résultat attendu :

| Paramètre | Valeur obtenue |
|---|---|
| IPv4 | 172.16.10.12 |
| Passerelle | 172.16.10.254 |
| Serveur DHCP | 172.16.10.1 |
| DNS | 172.16.10.1 |
| Suffixe DNS | tssr.lan |

> **Incident réseau rencontré (rappel)** : si la machine reçoit une adresse APIPA (169.254.x.x), voir la section "Dépannage DHCP" du fichier `CONFIGURATION.md`.

---

## 5. Vérification finale de l'installation

- [ ] GLPI accessible via `http://172.16.10.5` depuis n'importe quel poste du LAN
- [ ] Connexion possible avec les comptes par défaut (`glpi`) et les comptes créés
- [ ] Aucun message d'erreur Apache/PHP dans les logs (`/var/log/apache2/error.log`)
- [ ] Dossier `install/` supprimé du serveur

La suite du paramétrage fonctionnel (entité, catégories, utilisateurs, parc) est détaillée dans **`CONFIGURATION.md`**.

