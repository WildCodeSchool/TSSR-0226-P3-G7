# Intégration clients

---

## 1. Création des VM CLIWIN01 et CLIWIN02

Les deux postes clients ont été créés sous VirtualBox avec la configuration suivante :

| Paramètre | CLIWIN01 | CLIWIN02 |
|---|---|---|
| OS | Windows 11 | Windows 11 |
| RAM | 4 Go | 4 Go |
| Disque | 60 Go (VDI) | 60 Go (VDI) |
| Réseau | LAN-Pharmgreen (interne) | LAN-Pharmgreen (interne) |
| Nom de machine | CLIWIN01 | CLIWIN02 |

L'installation de Windows 11 a été réalisée sans compte Microsoft en ligne, via la commande `oobe\bypassnro` lancée depuis l'invite de commandes accessible pendant la phase de configuration initiale (Maj+F10), permettant de créer un compte local temporaire.

---

## 2. Configuration réseau initiale

Aucune configuration IP statique n'est nécessaire côté client : les deux postes obtiennent automatiquement leur adresse via le serveur DHCP de SRVWIN01 (scope LAN-Pharmgreen, plage 172.16.10.10 -> 200).

Adresses obtenues et confirmées :

| Poste | Adresse IP | Bail DHCP |
|---|---|---|
| CLIWIN01 | 172.16.10.12 | Attribué par 172.16.10.1 |
| CLIWIN02 | 172.16.10.11 | Attribué par 172.16.10.1 |

---

## 3. Intégration au domaine tssr.lan

Sur chaque poste, en tant qu'administrateur local :

1. **Paramètres > Système > À propos > Renommer ce PC (options avancées)**
2. Onglet **Nom de l'ordinateur** > **Modifier**
3. Sélectionner **Domaine**, saisir `tssr.lan`
4. Fournir les identifiants d'un compte disposant des droits de jonction au domaine (compte administrateur du domaine)
5. Confirmer le message de bienvenue dans le domaine `tssr.lan`
6. Redémarrer le poste

**Point de vigilance identifié** : après jonction, un ordinateur est placé par défaut dans le conteneur **Computers** de l'Active Directory, et non dans l'unité d'organisation `OU=_Ordinateurs,OU=Pharmgreen`. Or, seule cette dernière reçoit l'application des GPO du Sprint 04. Un déplacement manuel de l'objet ordinateur est donc nécessaire après chaque jonction :

Sur **SRVWIN01**, dans `dsa.msc` (Utilisateurs et ordinateurs Active Directory) :
1. Localiser l'objet ordinateur dans **Computers**
2. Glisser-déposer (ou couper/coller) vers `OU=_Ordinateurs,OU=Pharmgreen,DC=tssr,DC=lan`

Ce déplacement a été réalisé pour CLIWIN01 et CLIWIN02.

---

## 4. Première connexion avec un compte de domaine

Une fois le poste redémarré et rejoint au domaine, la connexion s'effectue à l'écran de verrouillage via **Autre utilisateur**, en saisissant :

- Nom d'utilisateur : `tssr\<prenom.nom>` (ou `<prenom.nom>@tssr.lan`)
- Mot de passe : mot de passe temporaire défini à l'import (`Azerty1*`), avec changement obligatoire à la première connexion

À la première connexion, Windows crée automatiquement le profil utilisateur local (`C:\Users\<prenom.nom>`).

Comptes utilisés pour la validation :
- CLIWIN01 : `valentina.ferrari`
- CLIWIN02 : `pierre.david`

---

## 5. Application forcée des GPO après jonction

Après déplacement dans la bonne OU, une application forcée des stratégies de groupe a été effectuée sur chaque poste afin de ne pas attendre le cycle de rafraîchissement automatique (90 minutes par défaut) :

```powershell
gpupdate /force
```

