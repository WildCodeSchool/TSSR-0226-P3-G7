# INSTALLATION - IPBX01 (FreePBX)





------------------------------------------------------------------------------------------------------------

# GUIDE INSTALLATION PAS A PAS: IPBX01 (FreePBX)

## 1. Création de la VM (T01)

**Hyperviseur** : VirtualBox 7.x (assistant de création "tout-en-un")

### 1.1 Étape "Virtual machine name and operating system"

Ouvrir VirtualBox Manager -> **Nouvelle**, puis renseigner :

| Champ | Valeur |
|---|---|
| VM Name | IPBX01 |
| VM Folder | dossier VirtualBox VMs habituel |
| ISO Image | `SNG7-PBX16-64bit-2302-1.iso` |
| OS | Linux |
| OS Distribution | Red Hat |
| OS Version | Red Hat (64-bit) |
| Proceed with Unattended Installation | **décoché** |

> **Point de vigilance sur l'ISO** : utiliser impérativement **FreePBX 16** (`SNG7-PBX16-64bit-2302-1.iso`, base CentOS 7). L'ISO **FreePBX 17** (`SNGDEB-PBX17-*`, base Debian) génère des échecs de boot (capture clavier GRUB, erreur de détection de taille disque, module LVM absent de l'initrd) et est à proscrire pour ce projet.
>
> Ne pas cocher "Proceed with Unattended Installation" , cette option interfère avec l'installeur natif de SangomaOS ; on effectue une installation manuelle classique.

### 1.2 Étape "Specify virtual hardware"

| Champ | Valeur |
|---|---|
| Base Memory | 2048 Mo |
| Number of CPUs | 2 |
| Use EFI | décoché |

### 1.3 Étape "Specify virtual hard disk"

| Champ | Valeur |
|---|---|
| Mode | Create a New Virtual Hard Disk |
| Disk Size | 30 Go |
| Format | VDI (VirtualBox Disk Image) |
| Pre-allocate Full Size | **décoché** (allocation dynamique) |
| Split Disk Into 2 GB Parts | décoché |

Cliquer sur **Finish** pour créer la VM.

### 1.4 Réglages complémentaires avant premier démarrage

Clic droit sur IPBX01 -> **Configuration** :

- **Stockage** : vérifier que l'ISO `SNG7-PBX16-64bit-2302-1.iso` est bien montée sur le contrôleur IDE
- **Réseau** -> Adaptateur 1 :
  - Activer la carte réseau : coché
  - Mode d'accès réseau : **Réseau interne**
  - Nom : **`LAN-Pharmgreen`** (réseau interne commun à SRVWIN01, SRVLX01, SRVWIN04, FW01)

---

## 2. Installation de FreePBX (T02)

### 2.1 Démarrage et menu GRUB

Démarrer la VM. Au menu GRUB (SangomaOS 7.8), sélectionner à l'aide des flèches :
```
FreePBX 16 Installation (Asterisk 18) - Recommended
```
puis valider avec **Entrée**.

### 2.2 Sous-menu de méthode d'installation

Un second menu apparaît, propre à l'installeur Sangoma :
```
Graphical Installation - Output to VGA
```
*(premier choix, déjà pré-sélectionné)* : c'est l'option adaptée à un pilotage via la fenêtre VirtualBox.

### 2.3 Chargement et écran de configuration Sangoma

Après chargement du noyau (1 à 3 minutes, défilement de logs), l'écran graphique **"CONFIGURATION - USER SETTINGS"** s'affiche avec deux blocs :

- **ROOT PASSWORD** : cliquer dessus, saisir le mot de passe root, **Done**
- **USER CREATION** : purement informatif (l'utilisateur système `asterisk` est créé automatiquement), aucune action requise

L'installation des paquets démarre en arrière-plan dès l'affichage de cet écran (barre de progression visible en bas, ex. `Installing asterisk-sounds-moh-opsound-alaw (678/739)`).

### 2.4 Fin d'installation

Une fois tous les paquets installés et la génération de l'initramfs terminée, l'écran affiche :
```
Complete!
SangomaOS 7.8 is now successfully installed and ready for you to use!
```

**Avant de redémarrer** : éjecter l'ISO du lecteur virtuel (**Périphériques -> Lecteurs optiques -> Retirer le disque du lecteur virtuel**), pour éviter un nouveau boot sur le menu d'installation.

Cliquer ensuite sur **Reboot**.

### 2.5 Premier démarrage

Le système redémarre sur l'invite de connexion console :
```
Sangoma Linux 7 (Core) (x86_64)
Kernel version 3.10.0-1127.19.1.el7.x86_64

freepbx login:
```

Se connecter avec `root` et le mot de passe défini à l'étape 2.3. Un bandeau d'accueil FreePBX s'affiche, résumant la configuration réseau détectée (interface `eth0`, adresse IP obtenue par DHCP à ce stade , sera fixée en IP statique à l'étape 3).

### 2.6 Post-installation - clavier console

Par défaut, le clavier console est en `us` (QWERTY). Pour passer en AZERTY français :
```bash
localectl set-keymap fr
```
Vérification :
```bash
localectl status
# doit afficher : VC Keymap: fr / X11 Layout: fr
```

---

## 3. Configuration IP statique (T03)

Fichier : `/etc/sysconfig/network-scripts/ifcfg-eth0`

Modifications apportées :
```
BOOTPROTO="none"
IPADDR="172.16.10.6"
PREFIX="24"
GATEWAY="172.16.10.254"
DNS1="172.16.10.1"
```

Application :
```bash
systemctl restart network
```

Vérification :
```bash
ip a                        # confirme inet 172.16.10.6/24 sur eth0
ping  172.16.10.254         # confirme la connectivité vers la passerelle
```

---

## 4. Accès à l'interface web

Depuis un poste du réseau `172.16.10.0/24` (CLIWIN01, CLIWIN02) se connecter à:
```
http://172.16.10.6
```

### 5.1 Écran "Initial Setup"

Premier écran affiché à la connexion sur `http://172.16.10.6` :

| Champ | Valeur |
|---|---|
| Username | admin |
| Password | Azerty1* |
| Confirm Password | Azerty1* |
| Notifications Email address | admin@pharmgreen.fr |
| System Identifier | IPBX01 *(remplace la valeur par défaut "VoIP Server")* |
| Automatic Module Updates | Disabled |
| Automatic Module Security Updates | Enabled *(par défaut)* |
| Send Security Emails For Unsigned Modules | Enabled *(par défaut)* |
| Check for Updates every / Between | Saturday / 8am-12pm *(par défaut, sans incidence)* |

Cliquer sur **Setup System**.

### 5.2 Assistant Sangoma Smart Firewall (5 écrans successifs)

Cet assistant s'enchaîne automatiquement après le Setup System :

1. **"Sangoma Smart Firewall is now enabled!"** -> cliquer sur **Continue**
2. **"Should the client you're using be trusted?"** (détecte l'IP du poste d'administration, ici 172.16.10.12 / CLIWIN01) -> **Yes** *(évite tout risque de blocage accidentel de l'administrateur)*
3. **"Enable Responsive Firewall?"** -> **No** *(fonctionnalité destinée aux clients SIP distants/dynamiques hors réseau local ; tous nos softphones sont internes avec IP fixe, donc non nécessaire)*
4. **"Automatically configure Asterisk IP Settings?"** -> **Yes** *(réseau simple à passerelle unique, cas d'usage standard recommandé par l'assistant)*
5. **"SIPStation SIP Trunks Free Trial"** (offre commerciale de trunk téléphonique externe) -> **Not Now** *(hors périmètre : le sprint ne couvre que la téléphonie interne)*

### 5.3 Écran final : Dashboard

L'assistant se termine sur le tableau de bord FreePBX (**Vue d'ensemble système**), confirmant l'état des services : Asterisk, MySQL, Serveur Web, Fail2Ban, Firewall Configuration, System Firewall, Mail Queue, UCP Daemon, Xmpp Daemon -> tous doivent apparaître avec une coche verte.

