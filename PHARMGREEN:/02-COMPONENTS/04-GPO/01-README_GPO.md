# GPO : Stratégies de Groupe
> Documentation technique de la brique GPO du projet Pharmgreen.
> Couvre les 7 GPO obligatoires créées et testées durant le Sprint 04.

---

## 1. Informations générales

| Paramètre | Valeur |
|---|---|
| Serveur | SRVWIN01 |
| Console | Group Policy Management Console (gpmc.msc) |
| Domaine | tssr.lan |
| Nombre de GPO créées | 7 |
| Sprint | Sprint 04 |

---

## 2. Contenu du dossier

| Fichier | Description |
|---|---|
| `README.md` | Ce fichier: présentation de la brique GPO |
| `configuration.md` | Configuration détaillée de chaque GPO |

---

## 3. Liste des GPO

| Tâche | Nom de la GPO | Objectif | Liée à |
|---|---|---|---|
| T01 | GPO-DOM-PasswordPolicy-01 | Politique de mot de passe | tssr.lan |
| T02 | GPO-DOM-AccountLockout-01 | Verrouillage de compte | tssr.lan |
| T03 | GPO-DOM-ControlPanel-Block-01 | Blocage panneau de configuration | OU=_Utilisateurs |
| T04 | GPO-DOM-LocalAdmin-01 | Gestion des admins locaux | OU=_Ordinateurs |
| T05 | GPO-DOM-PowerShell-Security-01 | Sécurisation PowerShell | OU=_Ordinateurs |
| T06 | GPO-DOM-Custom-01-Fond d'écran Pharmgreen | Fond d'écran personnalisé | OU=_Utilisateurs |
| T07 | GPO-DOM-Custom-02-Désactiver clé USB | Blocage des clés USB | OU=_Ordinateurs |

---

## 4. Logique d'application des GPO

<img width="725" height="421" alt="LOGIQUE_GPO" src="https://github.com/user-attachments/assets/4035e2fc-1b32-4f1b-8363-0415d23e9d8a" />


---

## 5. Liens

- [configuration.md](configuration.md)
- [active-directory/configuration.md](../active-directory/configuration.md)

