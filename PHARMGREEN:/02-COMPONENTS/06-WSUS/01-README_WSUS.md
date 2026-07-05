# WSUS : MISE A JOUR 

---

## 1. Objectif du sprint

Déployer un serveur **WSUS** (Windows Server Update Services) afin de centraliser et contrôler la distribution des mises à jour Windows sur l'ensemble du parc Pharmgreen, mettant fin à des mises à jour gérées de façon anarchique poste par poste.

---

## 2. Architecture

| Élément | Valeur |
|---|---|
| **Serveur** | SRVWIN04 |
| **Système d'exploitation** | Windows Server 2022 |
| **Adresse IP** | 172.16.10.4 (statique) |
| **Masque** | 255.255.255.0 (/24) |
| **Passerelle** | 172.16.10.254 (FW01 / pfSense) |
| **DNS** | 172.16.10.1 (SRVWIN01) |
| **Réseau virtuel** | LAN-Pharmgreen (Réseau interne VirtualBox) |
| **Rôle** | Windows Server Update Services (WSUS) |
| **Dossier de contenu** | C:\WSUS |
| **Domaine** | tssr.lan (Member Server) |


---

## 3. Documents du sprint

| Fichier | Contenu |
|---|---|
| **README.md** (ce fichier) | Vue d'ensemble, objectifs, architecture |
| **INSTALL.md** | Procédure de préparation de la VM et installation du rôle WSUS |
| **CONFIGURATION.md** | Paramétrage fonctionnel (groupes de machines, approbations) et incident de synchronisation rencontré |

---

## 4. Récapitulatif des tâches du sprint

| Tâche | Description | Statut |
|---|---|---|
| T01 | Créer VM SRVWIN04 | Terminée (via clonage de template) |
| T02 | Installer Windows Server 2022 | Terminée (héritée du clone) |
| T03 | Configurer IP statique 172.16.10.4 | Terminée |
| T04 | Installer et configurer rôle WSUS | Terminée |
| T05 | Créer groupes de machines WSUS | Terminée |
| T06 | Configurer approbation MAJ de sécurité | Partielle - configuration prête, non sauvegardée (voir CONFIGURATION.md) |
| T07 | Synchroniser WSUS avec Microsoft Update | **Bloquée** - incident externe côté Microsoft (voir CONFIGURATION.md, section 4) |
| T08 | Tester déploiement MAJ sur CLIWIN01 | En attente de la levée du blocage T07 |
| T09 | Documenter wsus/ | Terminée (ce document) |

---

## 5. Point d'attention majeur

La synchronisation initiale avec Microsoft Update échoue actuellement pour une cause **externe à l'infrastructure Pharmgreen** : un incident documenté côté serveurs de catalogue Microsoft, affectant de nombreux administrateurs WSUS depuis mi-juin 2026, indépendamment de la version de Windows Server utilisée. Le détail technique et le suivi de cet incident sont dans `CONFIGURATION.md`.

**Ce sprint reste donc partiellement bloqué pour une raison hors du contrôle du prestataire**, et sera repris dès que la situation sera résolue côté Microsoft.

---

## 6. Points ouverts / améliorations futures

- Relancer périodiquement la synchronisation WSUS (T07) jusqu'à résolution de l'incident Microsoft
- Une fois la synchronisation réussie : finaliser T06 (sauvegarde de la règle d'approbation automatique), puis T08 (test de déploiement sur CLIWIN01)
- Envisager, si le blocage perdure trop longtemps, une stratégie alternative temporaire (ex : approbation manuelle ciblée sur un sous-ensemble de mises à jour déjà en base, si disponibles)

---
