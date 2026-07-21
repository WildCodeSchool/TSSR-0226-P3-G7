# Sprint08: Intégration clients & tests

---

## 1. Objectifs du sprint

Ce sprint vise à intégrer les postes clients Windows au domaine `tssr.lan` et à valider, de bout en bout, le bon fonctionnement de l'ensemble de l'infrastructure déployée lors des sprints précédents (Active Directory, GPO, DHCP/DNS, VoIP, messagerie) du point de vue de l'utilisateur final.

Deux postes clients ont été déployés :

| Poste | OS | IP | Compte principal testé |
|---|---|---|---|
| CLIWIN01 | Windows 11 | 172.16.10.12 | valentina.ferrari |
| CLIWIN02 | Windows 11 | 172.16.10.11 | pierre.david |

---

## 2. Architecture concernée

Ce sprint ne crée pas de nouvelle brique d'infrastructure : il **valide** l'intégration des postes clients avec les services déjà en place :

- **SRVWIN01** (172.16.10.1) : Active Directory, DNS, DHCP, GPO
- **SRVLX01** (172.16.10.5) : GLPI, messagerie iRedMail
- **IPBX01** (172.16.10.6) : VoIP FreePBX

---

## 3. Documents du sprint

| Fichier | Contenu |
|---|---|
| **README.md** (ce fichier) | Vue d'ensemble, objectifs, récapitulatif des tâches |
| **INSTALL.md** | Création des VM CLIWIN01/CLIWIN02 et intégration au domaine |
| **CONFIGURATION.md** | Détail des tests fonctionnels (GPO, connexion AD, DHCP/DNS, VoIP/messagerie et modèle AGDLP) |

---

## 4. Récapitulatif des tâches du sprint

| Tâche | Description | Statut |
|---|---|---|
| T01 | Créer VM CLIWIN01 Windows 11 (4 Go RAM, 60 Go) | Terminée |
| T02 | Créer VM CLIWIN02 Windows 11 (4 Go RAM, 60 Go) | Terminée |
| T03 | Intégrer CLIWIN01 au domaine tssr.lan | Terminée |
| T04 | Intégrer CLIWIN02 au domaine tssr.lan | Terminée |
| T05 | Vérifier application des 7 GPO sur les clients | Partielle - 5/7 confirmées (voir T05) |
| T06 | Tester connexion utilisateur AD depuis les clients | Terminée |
| T07 | Valider DHCP, DNS et résolution de noms | Terminée |
| T08 | Valider VoIP (appel 3CX) et messagerie iRedMail | Terminée |
| T09 | Documenter clients/ | Terminée (ce document) |

---

## 5. Point d'attention majeur : T05 (7 GPO)

| # | GPO | Statut |
|---|---|---|
| 1 | GPO-DOM-PasswordPolicy-01 | Confirmée |
| 2 | GPO-DOM-AccountLockout-01 | Confirmée |
| 3 | GPO-DOM-ControlPanel-Block-01 | Confirmée |
| 4 | GPO-DOM-Custom-01 (fond d'écran) | Validée et corrigée |
| 5 | GPO-DOM-LocalAdmin-01 |  **Anomalie non résolue** |
| 6 | GPO-DOM-PowerShell-Security-01 | Confirmée (AllSigned) |
| 7 | GPO-DOM-Custom-02 (USB) |  **Non testée** - bloquée |

**GPO-DOM-LocalAdmin-01** : la configuration côté GPO est correcte (`Members of this group` = `TSSR\Domain Admins`, portée confirmée "winning" via `gpresult /h`), et la GPO est bien appliquée sur CLIWIN01. Malgré cela, le compte local intégré `CLIWIN01\Administrator` reste membre du groupe Administrateurs local. Testé sans succès : `gpupdate /force`, redémarrage complet de la machine, réinitialisation du cache de version de la CSE Security (`Extension-List\{827D319E-...}`). Cause exacte non identifiée à ce stade. Point mis en pause pour ne pas bloquer la progression du sprint ; à reprendre ultérieurement (piste à explorer : comparaison `net localgroup` vs `Get-LocalGroupMember`, export `secedit` de la base SAM locale).

**GPO-DOM-Custom-02 (USB)** : test fonctionnel impossible en l'état, faute de clé USB physique disponible pour un passthrough vers la VM CLIWIN01/02 via VirtualBox.

---

## 6. Point d'attention complémentaire : modification du modèle AGDLP

En préparant le test T06, il a été constaté que les 208 comptes utilisateurs importés en Sprint 03 avaient bien été placés dans leurs OU de département respectives, mais **n'avaient jamais été affectés à leur groupe global (GG) correspondant**, et que les groupes GG n'étaient eux-mêmes jamais imbriqués dans les groupes locaux de domaine (GDL) de partage. Le modèle AGDLP prévu par l'architecture (11 GG + 6 GDL) était donc créé mais inopérant.

Ce point a été corrigé au cours de ce sprint (détail complet en `CONFIGURATION.md`) :
- Affectation des 208 utilisateurs à leur GG de département
- Imbrication des 11 GG dans `GDL-Partages-Lecture` et `GDL-Partages-Ecriture`

Le cloisonnement réel des droits d'écriture par département (ACL NTFS sur des sous-dossiers dédiés) est reporté au **Sprint 09**, au moment de la création effective des dossiers partagés.

---

## 7. Points ouverts / améliorations futures

- Résoudre l'anomalie GPO-DOM-LocalAdmin-01 (compte administrateur local non retiré)
- Tester GPO-DOM-Custom-02 dès qu'une clé USB sera disponible pour passthrough VirtualBox
- Sprint 09 : créer l'arborescence des dossiers partagés par département et appliquer les ACL NTFS correspondantes

---
