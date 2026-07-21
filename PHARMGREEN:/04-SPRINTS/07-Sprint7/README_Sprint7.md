# VoIP & Messagerie

## 1. Description du sprint
Déploiement des services de communication interne de Pharmgreen : téléphonie sur IP (VoIP) et messagerie électronique professionnelle.

## 2. Objectifs
Doter Pharmgreen d'une solution de téléphonie IP fonctionnelle entre les postes du domaine, ainsi que d'une messagerie électronique sur son propre nom de domaine.

## 3. Comment
**VoIP :**
- Création d'IPBX01 (FreePBX 16 sur SangomaOS/CentOS 7, 172.16.10.6)
- Création de 22 extensions SIP couvrant les 11 départements (création manuelle de référence puis import en masse via Bulk Handler CSV)
- Installation du softphone 3CXPhone sur CLIWIN01 et CLIWIN02
- Test d'appel réussi entre les deux postes clients

**Messagerie :**
- Installation d'iRedMail 1.8.3 sur SRVLX01, domaine `pharmgreen.fr`
- Création de nouvelles boîtes mail via script (avec normalisation Unicode des noms), portant le total à 209 boîtes
- Migration de GLPI sur le port 8080 pour lever un conflit Apache/Nginx introduit par l'installation d'iRedMail
- Configuration d'Outlook Classic sur CLIWIN01 et CLIWIN02 (IMAP 993 SSL/TLS, SMTP 587 STARTTLS)
- Test d'échange de courrier bidirectionnel réussi entre les deux postes

## 4. Résultats
Téléphonie VoIP et messagerie électronique pleinement opérationnelles et validées fonctionnellement entre les deux postes clients du domaine.
