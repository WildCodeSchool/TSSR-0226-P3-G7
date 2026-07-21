# Sprint 04 : Stratégies de groupe (GPO)

## 1. Description du sprint
Sécurisation et standardisation de l'environnement de travail des utilisateurs et des postes du domaine via la mise en œuvre de stratégies de groupe (GPO).

## 2. Objectifs
Mettre en œuvre et administrer de stratégies de groupe dans un domaine Windows et appliquer un socle de sécurité de base cohérent avec les pratiques d'entreprise.

## 3. Comment
Création de 7 GPO via `gpmc.msc` sur SRVWIN01 :
- Liées à la racine `tssr.lan` : `GPO-DOM-PasswordPolicy-01`, `GPO-DOM-AccountLockout-01`
- Liées à `OU=_Utilisateurs` : `GPO-DOM-ControlPanel-Block-01`, `GPO-DOM-Custom-01` (fond d'écran imposé)
- Liées à `OU=_Ordinateurs` : `GPO-DOM-LocalAdmin-01`, `GPO-DOM-PowerShell-Security-01`, `GPO-DOM-Custom-02` (blocage clé USB)

Premiers tests réalisés sur CLIWIN02, nécessitant le déplacement du poste depuis le conteneur `Computers` par défaut vers `OU=_Ordinateurs`.

## 4. Résultats
Les 7 GPO ont été créées et liées avec succès. La validation fonctionnelle complète (comportement réel sur les postes clients) a été réalisée ultérieurement, au Sprint 08 ; voir ce sprint pour le détail des tests et l'anomalie identifiée sur `GPO-DOM-LocalAdmin-01`.

