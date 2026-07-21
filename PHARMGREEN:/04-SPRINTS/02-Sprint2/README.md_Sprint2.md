# Sprint 02 : Pare-feu FW01 (pfSense)

## 1. Description du sprint
Déploiement du pare-feu périmétrique FW01 sous pfSense CE, point d'entrée et de cloisonnement de l'ensemble de l'infrastructure réseau Pharmgreen.

## 2. Objectifs
Sécuriser la frontière réseau de l'infrastructure et établir une séparation effective entre les zones LAN, DMZ et WAN.

## 3. Comment
- Création de la VM FW01 (VirtualBox, 1 Go RAM, 20 Go disque, 3 interfaces réseau)
- Installation de pfSense CE 2.8.1
- Attribution des interfaces : WAN (NAT), LAN-Pharmgreen (172.16.10.254/24), DMZ-Pharmgreen (172.16.20.254/24)
- Configuration de 4 règles de pare-feu :
  - R01 : autorisation LAN -> any (par défaut)
  - R02 : autorisation WAN -> DMZ, ports 80/443
  - R03 : blocage explicite DMZ -> LAN
  - R04 : deny all implicite (règle de clôture)
- Tests de connectivité depuis un poste client : ping passerelle, ping externe (8.8.8.8), `tracert` confirmant FW01 comme premier saut

## 4. Résultats
FW01 opérationnel, connectivité LAN/WAN validée, cloisonnement DMZ->LAN confirmé bloqué. Le pare-feu constitue la base réseau de tous les sprints suivants.

