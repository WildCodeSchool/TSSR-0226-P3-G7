# Sprint 03 : Active Directory, DNS, DHCP

## 1. Description du sprint
Déploiement du contrôleur de domaine Active Directory et des services réseau associés (DNS, DHCP), structuration de l'annuaire et import des utilisateurs de l'entreprise.

## 2. Objectifs
Établir le socle d'identité (comptes, groupes, OU) et de services réseau fondamentaux (résolution de noms, attribution IP) de l'infrastructure Pharmgreen.

## 3. Comment
- Promotion de SRVWIN01 (Windows Server 2025, 172.16.10.1) en contrôleur de domaine, création de la forêt `tssr.lan`
- Configuration DNS avec forwarders 8.8.8.8 et 1.1.1.1
- Configuration DHCP : scope LAN-Pharmgreen 172.16.10.10–200, désactivation du DHCP natif de pfSense
- Création de l'arborescence OU complète : `Pharmgreen` > `_Utilisateurs` (11 OU de département), `_Ordinateurs`, `_Groupes`, `_Admins`, `_Serveurs`
- Création des 11 groupes globaux (GG) par département et des 6 groupes locaux de domaine (GDL) selon le modèle AGDLP
- Import de 208 utilisateurs via le script `import-users.ps1` (filtrage `Société = "Pharmgreen"`, gestion de l'encodage Windows-1252 et des incohérences de nommage du CSV source)

## 4. Résultats
Domaine `tssr.lan` pleinement opérationnel, 208 comptes utilisateurs créés et correctement répartis dans leurs OU de département. Structure de groupes AGDLP créée mais non peuplée à ce stade;le rattachement effectif des utilisateurs aux groupes GG/GDL a été réalisé lors du Sprint 08, à la suite d'un contrôle de cohérence.

