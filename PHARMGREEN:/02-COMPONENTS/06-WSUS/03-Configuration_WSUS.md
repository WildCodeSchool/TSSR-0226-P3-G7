# PARAMETRAGE WSUS

> Ce document suppose que le rôle WSUS est déjà installé sur SRVWIN04 (voir `INSTALL.md`). Il couvre le paramétrage fonctionnel réalisé, ainsi que le diagnostic détaillé de l'incident de synchronisation rencontré durant ce sprint.

---

## 1. Groupes de machines WSUS

**Chemin** : Console WSUS -> **Computers -> All Computers -> Add Computer Group...**

Deux groupes ont été créés pour organiser le parc Pharmgreen :

| Groupe | Usage prévu |
|---|---|
| **Serveurs** | Machines serveurs Windows Server (ex : futurs serveurs à intégrer) |
| **Postes clients** | Postes Windows 11 (CLIWIN01, CLIWIN02) |

**Procédure** :
1. Clic droit sur **All Computers** -> **Add Computer Group...**
2. Nom du groupe (`Serveurs` ou `Postes clients`)
3. **Add**

**Résultat** : les deux groupes apparaissent sous `All Computers`, aux côtés du groupe par défaut `Unassigned Computers`.

---

## 2. Règle d'approbation automatique (T06)

**Chemin** : Console WSUS -> **Options -> Automatic Approvals**

**Règle par défaut identifiée** :

> Quand une mise à jour est de type **Critical Updates** ou **Security Updates** -> Approuver pour **tous les ordinateurs**

** Statut à date de rédaction** : cette règle n'a pas pu être **sauvegardée**, la console WSUS affichant le message :

```
Cannot save configuration because the server is synchronizing.
```

Ce blocage est directement lié à l'incident de synchronisation détaillé en section 4 ci-dessous - WSUS verrouille certains paramètres tant qu'une synchronisation est active ou en tentative répétée. **Cette tâche sera finalisée dès que le blocage de synchronisation sera levé.**

---

## 3. Tentative de synchronisation avec Microsoft Update (T07)

### 3.1 Procédure suivie

1. Lancement de l'assistant de configuration WSUS (Gestionnaire de serveur → Outils → Windows Server Update Services, premier lancement)
2. Avant de commencer : Microsoft Update Improvement Program → décoché
3. Choix du serveur amont : **Synchronize from Microsoft Update**
4. Proxy : aucun proxy configuré (accès direct via NAT FW01/pfSense)
5. Étape **"Connect to Upstream Server"** -> **Start Connecting**

### 3.2 Symptôme observé

La barre de progression reste bloquée indéfiniment (plusieurs tentatives de 15 à 45 minutes), sans jamais dépasser 0%, avec le statut `Synchronizing...` figé.

---

## 4. Diagnostic détaillé de l'incident de synchronisation

### 4.1 Éléments réseau et système vérifiés (tous concluants - non responsables du blocage)

| Vérification | Commande | Résultat |
|---|---|---|
| Connectivité HTTPS vers Microsoft | `Test-NetConnection www.microsoft.com -Port 443` | TcpTestSucceeded : True |
| Connectivité HTTPS générique | `Test-NetConnection google.com -Port 443` | TcpTestSucceeded : True |
| Résolution DNS serveur de sync WSUS | `Resolve-DnsName sws.update.microsoft.com` | Résolu normalement |
| Connectivité vers le serveur de sync | `Test-NetConnection sws.update.microsoft.com -Port 443` | TcpTestSucceeded : True |
| Proxy WinHTTP | `netsh winhttp show proxy` | Direct access (no proxy) |
| Pool d'application IIS | `Get-WebAppPoolState -Name "WsusPool"` | Started |
| Certificats racine Microsoft | `Get-ChildItem Cert:\LocalMachine\Root` | Tous valides, dates d'expiration lointaines |
| Service BITS | `Get-Service BITS` |  Trouvé arrêté (Stopped) -> redémarré manuellement, sans effet sur le blocage |
| Correctif TLS 1.2 .NET Framework | `SchUseStrongCrypto` | Appliqué (clé absente au départ) -> sans effet sur le blocage |

### 4.2 Cause racine identifiée

Consultation du journal détaillé :

```powershell
Get-Content "C:\Program Files\Update Services\LogFiles\SoftwareDistribution.log" -Tail 50 | Select-String -Pattern "error|fail|exception" -Context 2
```

Le journal révèle un flux massif d'erreurs répétées du type :

```
SqlException occurred. Number 50000 and message invalid update identity in XML for update {GUID}\100
SqlException occurred. Number 50000 and message invalid update identity (AtLeastOne Prerequisite) in XML for update {GUID}\100
```

Ainsi qu'un résultat de synchronisation final : **"An HTTP error occurred"**.

**Analyse** : ce signal correspond à un **incident documenté, externe à l'infrastructure Pharmgreen**, actuellement en cours sur les serveurs de catalogue de mises à jour de Microsoft. Le mécanisme est le suivant : lors de la synchronisation, WSUS télécharge des métadonnées de mise à jour depuis Microsoft ; ces métadonnées contiennent actuellement des structures XML malformées et des identités de mise à jour invalides ; les contrôles de sécurité internes de la base WSUS (SUSDB) rejettent ces données pour éviter une corruption, ce qui provoque l'erreur SQL 50000 en boucle et bloque la synchronisation indéfiniment à 0%.

Ce problème touche de nombreux administrateurs WSUS sur plusieurs versions de Windows Server (2016 à 2025), indépendamment de leur configuration locale, et semble être apparu à partir de la mi-juin 2026. Les correctifs purement locaux (réinstallation du rôle, recréation de la base SUSDB, réduction du périmètre des produits/classifications) ne résolvent pas ce type de blocage, puisque la cause se situe côté catalogue Microsoft.

**Piste de contournement rapportée par d'autres administrateurs** : décocher le produit "Windows Insider Dev Channel" dans **Options -> Products and Classifications**. Cette piste n'a pas pu être appliquée dans notre cas : la liste des produits WSUS reste elle-même incomplète tant que la synchronisation initiale échoue, et ce produit n'apparaît pas encore dans la liste disponible.

### 4.3 Décision retenue pour ce sprint

Compte tenu de la nature externe et documentée du blocage :

1. **Aucune reconstruction de VM n'a été effectuée** - l'infrastructure locale (réseau, rôle WSUS, IIS, certificats, base de données) a été validée comme saine à chaque étape du diagnostic.
2. La tâche T07 est marquée **bloquée (cause externe)** plutôt qu'en échec côté prestataire.
3. Une nouvelle tentative de synchronisation sera relancée périodiquement, sans action supplémentaire nécessaire de notre part, dans l'attente d'une résolution côté Microsoft.
4. Les tâches T06 et T08, dépendantes d'une synchronisation réussie, sont mises en attente.

---

## 5. Suivi de l'incident

| Date | Action | Résultat |
|---|---|---|
| Sprint 06 | 6 tentatives de synchronisation manuelle | Toutes échouées ou annulées, blocage à 0% |
| Sprint 06 | Diagnostic complet (réseau, IIS, certificats, BITS, TLS) | Infrastructure locale confirmée saine |
| Sprint 06 | Analyse des logs SoftwareDistribution.log | Cause identifiée : incident de catalogue côté Microsoft |
| À planifier | Nouvelle tentative de synchronisation | En attente |

---

## 6. Prochaines étapes une fois le blocage levé

1. Relancer **Synchronize Now** dans la console WSUS
2. Vérifier que la progression dépasse 0% et que le statut passe à `Succeeded`
3. Sauvegarder la règle d'approbation automatique (T06)
4. Approuver manuellement quelques mises à jour critiques pour test si nécessaire
5. Déployer et tester la réception des mises à jour sur **CLIWIN01** (T08) :
   ```powershell
   gpupdate /force
   wuauclt /detectnow
   ```
6. Vérifier dans la console WSUS que CLIWIN01 apparaît dans **Unassigned Computers**, puis l'assigner au groupe **Postes clients**
