# WSUS (mises à jour centralisées)

## 1. Description du sprint
Déploiement du service WSUS (Windows Server Update Services) pour centraliser la distribution des mises à jour de sécurité sur le parc Windows de Pharmgreen.

## 2. Objectifs
Permettre une gestion contrôlée et centralisée des mises à jour Windows, plutôt qu'une mise à jour individuelle poste par poste.

## 3. Comment
- Création de SRVWIN04 (Windows Server 2022, 172.16.10.4) par clonage d'un template existant
- Installation du rôle WSUS, contenu stocké sur `C:\WSUS`
- Création de deux groupes de machines : **Serveurs** et **Postes clients**
- Tentative de configuration de la règle d'approbation automatique des mises à jour de sécurité
- Diagnostic complet en cas de blocage de synchronisation (réseau, DNS, certificats racine, service BITS, clés de registre TLS 1.2)

## 4. Résultats
Rôle WSUS installé et configuré côté infrastructure. La synchronisation initiale avec Microsoft Update est restée bloquée en raison d'un **incident externe documenté côté catalogue Microsoft** (erreur SQL 50000, métadonnées XML malformées), affectant de nombreux administrateurs WSUS depuis mi-juin 2026; cause confirmée comme externe après un diagnostic local exhaustif ayant écarté toute origine interne à l'infrastructure Pharmgreen. Point en attente de résolution côté éditeur.

