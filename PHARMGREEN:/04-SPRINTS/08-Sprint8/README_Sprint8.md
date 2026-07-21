# Intégration clients & tests

## 1. Description du sprint
Intégration des postes clients Windows au domaine et validation fonctionnelle globale, de bout en bout, de l'ensemble des services déployés lors des sprints précédents.

## 2. Objectifs
Vérifier, du point de vue de l'utilisateur final, que l'infrastructure construite (AD, GPO, DHCP/DNS, VoIP, messagerie) fonctionne correctement en conditions réelles d'usage sur des postes clients joints au domaine.

## 3. Comment
- Création et jonction au domaine `tssr.lan` des postes CLIWIN01 et CLIWIN02, déplacement en `OU=_Ordinateurs`
- Vérification de l'application effective des 7 GPO du Sprint 04 sur les deux postes
- Tests de connexion avec des comptes de domaine standards (`valentina.ferrari`, `pierre.david`)
- Validation de l'attribution DHCP et de la résolution DNS interne/externe sur les deux postes
- Validation fonctionnelle de la VoIP et de la messagerie déjà déployées au Sprint 07
- Constat et correction du modèle AGDLP : affectation des 208 utilisateurs à leur groupe global (GG) de département, puis imbrication des 11 GG dans les groupes locaux de domaine `GDL-Partages-Lecture` et `GDL-Partages-Ecriture`

## 4. Résultats
- 5 GPO sur 7 confirmées pleinement fonctionnelles ; anomalie non résolue sur `GPO-DOM-LocalAdmin-01` (documentée) ; test de `GPO-DOM-Custom-02` (USB) non réalisable faute de matériel disponible
- Connexion AD, DHCP et DNS validées avec succès sur les deux postes clients
- Modèle AGDLP corrigé et rendu opérationnel (cloisonnement fin des droits d'écriture par ACL NTFS renvoyé au Sprint 09)

Documentation détaillée disponible dans `README.md`, `INSTALL.md` et `CONFIGURATION.md` du sprint.
