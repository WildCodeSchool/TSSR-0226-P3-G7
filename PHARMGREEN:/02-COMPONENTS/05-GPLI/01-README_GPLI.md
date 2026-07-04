# Gestion de parc GLPI

---

## 1. Objectif du sprint

Déployer et configurer **GLPI** (Gestion Libre de Parc Informatique) sur un serveur Linux dédié, afin de professionnaliser la gestion du parc matériel et le système de ticketing de l'entreprise Pharmgreen, dans le cadre de la mission de réorganisation de l'infrastructure IT confiée au prestataire.

---

## 2. Architecture

| Élément | Valeur |
|---|---|
| **Serveur** | SRVLX01 |
| **Système d'exploitation** | Debian 13 (Trixie) - CLI |
| **Adresse IP** | 172.16.10.5 (statique) |
| **Masque** | 255.255.255.0 (/24) |
| **Passerelle** | 172.16.10.254 (FW01 / pfSense) |
| **Réseau virtuel** | LAN-Pharmgreen (Réseau interne VirtualBox) |
| **Version GLPI** | 10.0.17 -> mise à jour 10.0.18 |
| **Stack applicative** | Apache2 + MariaDB + PHP 8.3 |

**Postes clients utilisés pour les tests d'accès :**

| Machine | OS | Rôle |
|---|---|---|
| CLIWIN01 | Windows 11 | Poste client de test (joint au domaine tssr.lan) |
| CLIWIN02 | Windows 11 | Poste client de test (joint au domaine tssr.lan) |

---

## 3. Documents du sprint

Ce sprint est documenté en trois fichiers complémentaires :

| Fichier | Contenu |
|---|---|
| **README.md** (ce fichier) | Vue d'ensemble, objectifs, architecture |
| **INSTALL.md** | Procédure d'installation pas à pas (LAMP, GLPI, postes clients) |
| **CONFIGURATION.md** | Paramétrage fonctionnel de GLPI (entité, catégories, utilisateurs, parc) et tests de validation |

---

## 4. Récapitulatif des tâches du sprint

| Tâche | Description | Statut |
|---|---|---|
| T01 | Créer VM SRVLX01 (Debian 13 CLI) | Terminée |
| T02 | Configurer IP statique 172.16.10.5 | Terminée |
| T03 | Installer Apache + MariaDB + PHP (LAMP) | Terminée |
| T04 | Installer et configurer GLPI | Terminée |
| T05 | Configurer la gestion de parc (inventaire matériel) | Terminée |
| T06 | Configurer le système de ticketing (catégories ITIL) | Terminée |
| T07 | Tester l'accès webGUI depuis un poste client (CLIWIN01) | Terminée |
| T08 | Documenter le sprint | Terminée |

---

## 5. Points ouverts / améliorations futures

- Créer les comptes GLPI restants pour Hassan Liffite et Rosalie Roux (Data scientists)
- Mettre en place la **synchronisation LDAP GLPI - Active Directory** (objectif secondaire du projet), afin que les 208 utilisateurs de l'AD soient automatiquement disponibles dans GLPI sans création manuelle
- Ajouter FW01 (pfSense) à l'inventaire du parc si une traçabilité complète de l'infrastructure réseau est souhaitée
- Appliquer la convention de modèles matériels (Dell OptiPlex / Latitude / PowerEdge) aux futurs postes physiques ajoutés au parc

