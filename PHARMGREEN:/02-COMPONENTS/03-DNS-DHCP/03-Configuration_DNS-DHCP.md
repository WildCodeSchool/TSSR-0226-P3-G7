# DNS & DHCP - Configuration
> Cette Documentation technique des rôles DNS et DHCP du projet couvre la zone directe `tssr.lan`, les `enregistrements A`, les forwarders et l'étendue DHCP.

---

## Présentation générale

> DNS : Le rôle DNS a été installé automatiquement avec AD DS lors de la promotion de SRVWIN01 en contrôleur de domaine. La zone directe tssr.lan a été créée automatiquement en même temps. Les forwarders 8.8.8.8 (Google) et 1.1.1.1 (Cloudflare) ont été configurés via le Gestionnaire DNS pour permettre la résolution des noms internet. Dix enregistrements A ont été créés manuellement pour référencer tous les serveurs et postes clients de l'infrastructure (FW01, SRVWIN01 à SRVWIN04, SRVLX01, IPBX01, CLIWIN01, CLIWIN02, SRVWEB01)


> DHCP : Le rôle DHCP a été installé via le Server Manager puis autorisé dans l'Active Directory avec le compte TSSR\Administrator. Le serveur DHCP de pfSense (FW01) a ensuite été désactivé pour laisser SRVWIN01 prendre le relais. L'étendue LAN-Pharmgreen a été créée avec la plage 172.16.10.10 -> 172.16.10.200, la passerelle 172.16.10.254 (FW01) et le serveur DNS 172.16.10.1 (SRVWIN01). Les adresses .1 → .9 sont hors plage et réservées aux serveurs en IP statique.

> Test de Validation : Le test depuis CLIWIN02 a confirmé que le DHCP distribue correctement les adresses depuis SRVWIN01 (172.16.10.1) et non plus depuis FW01, avec le suffixe DNS tssr.lan bien présent.


## 1. Informations générales

| Paramètre | Valeur |
|---|---|
| Serveur | SRVWIN01 |
| Adresse IP | 172.16.10.1 |
| Rôle DNS | Serveur DNS principal du domaine tssr.lan |
| Rôle DHCP | Serveur DHCP unique du réseau LAN |
| Système d'exploitation | Windows Server 2025 |
| Sprint | Sprint 03 |

---

## 2. Configuration DNS

### 2.1 Zones DNS

| Zone | Type | Intégration AD | Statut |
|---|---|---|---|
| tssr.lan | Primary | Active Directory-Integrated | Running |
| _msdcs.tssr.lan | Primary | Active Directory-Integrated | Running |

### 2.2 Enregistrements A - Zone tssr.lan

| Nom | Type | Adresse IP | Description |
|---|---|---|---|
| srvwin01 | A | 172.16.10.1 | DC principal - AD DS, DNS, DHCP |
| FW01 | A | 172.16.10.254 | Pare-feu pfSense |
| SRVWIN02 | A | 172.16.10.2 | DC secondaire (Sprint 09) |
| SRVWIN03 | A | 172.16.10.3 | DC tertiaire (Sprint 09) |
| SRVWIN04 | A | 172.16.10.4 | Serveur WSUS (Sprint 06) |
| SRVLX01 | A | 172.16.10.5 | Serveur GLPI + Messagerie (Sprints 05/07) |
| IPBX01 | A | 172.16.10.6 | Serveur VoIP FreePBX (Sprint 07) |
| CLIWIN01 | A | 172.16.10.10 | Poste client Windows 10 |
| CLIWIN02 | A | 172.16.10.11 | Poste client Windows 11 |
| SRVWEB01 | A | 172.16.20.1 | Serveur Web DMZ (Sprint 09) |

### 2.3 Forwarders DNS

| Adresse IP | Fournisseur | Usage |
|---|---|---|
| 172.16.10.254 | pfSense FW01 | Redirection vers internet |
| 8.8.8.8 | Google DNS | Résolution internet |
| 1.1.1.1 | Cloudflare DNS | Résolution internet (secondaire) |

### 2.4 Vérification DNS

```powershell
Get-DnsServerResourceRecord -ZoneName "tssr.lan" | Format-Table
Get-DnsServerForwarder
Resolve-DnsName srvwin01.tssr.lan
```

---

## 3. Configuration DHCP

### 3.1 Étendue LAN-Pharmgreen

| Paramètre | Valeur |
|---|---|
| Nom | LAN-Pharmgreen |
| Réseau | 172.16.10.0/24 |
| Plage de distribution | 172.16.10.10 -> 172.16.10.200 |
| Masque | 255.255.255.0 |
| Durée du bail | 8 jours |
| Statut | Actif |

### 3.2 Adresses réservées hors plage DHCP

| Adresse IP | Machine | Rôle |
|---|---|---|
| 172.16.10.1 | SRVWIN01 | DC principal |
| 172.16.10.2 | SRVWIN02 | DC secondaire |
| 172.16.10.3 | SRVWIN03 | DC tertiaire |
| 172.16.10.4 | SRVWIN04 | WSUS |
| 172.16.10.5 | SRVLX01 | GLPI + Messagerie |
| 172.16.10.6 | IPBX01 | VoIP |
| 172.16.10.254 | FW01 | Passerelle pfSense |

### 3.3 Options DHCP distribuées aux clients

| Option | Valeur | Description |
|---|---|---|
| 003 Router | 172.16.10.254 | Passerelle par défaut (FW01) |
| 006 DNS Servers | 172.16.10.1 | Serveur DNS (SRVWIN01) |
| 015 DNS Domain Name | tssr.lan | Suffixe DNS |

### 3.4 Vérification DHCP

```powershell
Get-DhcpServerv4Scope
Get-DhcpServerv4Lease -ScopeId 172.16.10.0
Get-DhcpServerv4OptionValue -ScopeId 172.16.10.0
```

---

## 4. Test de validation 

Test effectué depuis **CLIWIN02** après renouvellement du bail DHCP :

| Paramètre | Valeur obtenue | Statut |
|---|---|---|
| Adresse IPv4 | 172.16.10.10 | Dans la plage DHCP |
| Masque | 255.255.255.0 | ok |
| Passerelle | 172.16.10.254 | FW01 |
| Serveur DHCP | 172.16.10.1 | SRVWIN01 |
| Serveurs DNS | 172.16.10.1 | SRVWIN01 |
| Suffixe DNS | tssr.lan | ok |

---

## 5. Liens

- [ACTIVE-DIRECTORY/Configuration_DNS-DHCP.md](../active-directory/configuration_dns-dhcp.md)
- [firewall/configuration.md](../fw01/configuration.md)

