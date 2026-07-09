# INSTALLATION - IPBX01 (FreePBX)

## 1. Création de la VM (T01)

**Hyperviseur** : VirtualBox

| Paramètre | Valeur |
|---|---|
| Nom | IPBX01 |
| Type / Version | Linux / Red Hat (64-bit) |
| RAM | 2048 Mo |
| CPU | 2 vCPU |
| Disque | 30 Go, VDI, dynamiquement alloué |
| Réseau | Adaptateur 1 -> Réseau interne -> `LAN-Pharmgreen` |

---

## 2. Installation de FreePBX (T02)

**ISO utilisée** : `SNG7-PBX16-64bit-2302-1.iso` (FreePBX 16, base CentOS 7 / SangomaOS 7.8, Asterisk 18)

> Ne pas utiliser l'ISO FreePBX 17 (`SNGDEB-PBX17-*`, base Debian) - non compatible avec les paramétrages VirtualBox standards utilisés sur ce projet.

### Procédure

1. Démarrer la VM sur l'ISO montée (contrôleur IDE)
2. Menu de boot GRUB -> sélectionner **FreePBX 16 Installation (Asterisk 18) - Recommended**
3. Sous-menu -> **Graphical Installation - Output to VGA**
4. Écran de configuration Sangoma :
   - Définir le mot de passe **root**
   - Laisser l'utilisateur `asterisk` se créer automatiquement (géré par l'installeur)
5. Attendre la fin de l'installation (`Complete!`)
6. Éjecter l'ISO du lecteur virtuel (**Périphériques -> Lecteurs optiques -> Retirer le disque**)
7. **Reboot**

### Post-installation : clavier console

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

### Assistant de première configuration (Initial Setup)

| Champ | Valeur |
|---|---|
| Username | admin |
| Password | Azerty1* |
| System Identifier | IPBX01 |
| Automatic Module Updates | Disabled |
| Automatic Module Security Updates | Enabled |

### Sangoma Smart Firewall : choix retenus lors de l'assistant

| Question | Réponse | Justification |
|---|---|---|
| Activer le firewall | Continue | Bonne pratique de sécurité |
| Client actuel (172.16.10.12) de confiance | Yes | Poste d'administration légitime (CLIWIN01) |
| Enable Responsive Firewall | No | Uniquement des clients internes fixes, pas de clients distants dynamiques |
| Auto-configurer les IP Asterisk | Yes | Réseau simple, une seule passerelle |
| SIPStation Free Trial | Not Now | Service de trunk externe, hors périmètre du sprint |
| Activation système | Skip | Non nécessaire pour l'usage interne du lab |
