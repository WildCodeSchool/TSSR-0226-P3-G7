# Sprint 05 : GLPI (gestion de parc & ticketing)

## 1. Description du sprint
Déploiement de la solution GLPI pour la gestion du parc informatique et le ticketing incident/support de Pharmgreen.

## 2. Objectifs
Mettre à disposition un outil centralisé de gestion des actifs informatiques et de suivi des demandes/incidents utilisateurs.

## 3. Comment
- Création de SRVLX01 (Debian 13, 172.16.10.5) et installation d'une pile LAMP (Apache, MariaDB, PHP)
- Déploiement de GLPI 10.0.18, sécurisation post-installation (suppression de `install.php`, activation de `session.cookie_httponly`)
- Création de l'entité **Pharmgreen** et configuration de 9 catégories de tickets ITIL
- Création de 3 comptes nommés d'après le fichier RH : Tina Le Hecourt (Super-Admin), Cyril François et Béatrice Bonnet (Techniciens)
- Inventaire des VM de l'infrastructure dans le module de gestion de parc

## 4. Résultats
GLPI accessible et fonctionnel, testé avec succès depuis les postes clients du domaine. Structure de ticketing et gestion de parc opérationnelle et prête à l'usage.
