# Captures de configuration

Ce dossier regroupe les preuves visuelles du projet **TechLocalSARL-Network**, classées dans un ordre logique : configuration, vérifications, services réseau et sécurité.

## 1. Segmentation et VLAN

### Création des VLAN
![Création des VLAN](01-creation-vlan.png)

### Attribution des VLAN aux ports
![Attribution des VLAN aux ports](02-attribution-vlan-ports.png)

### Vérification des VLAN
![Vérification des VLAN](03-verification-vlan.png)

### Vérification des attributions
![Vérification des attributions](04-verification-attributions.png)

## 2. Trunk et routage inter-VLAN

### Configuration du trunk
![Configuration du trunk](05-mode-trunk.png)

### Vérification du trunk
![Vérification du trunk](06-verification-trunk.png)

### Configuration du routage inter-VLAN
![Routage inter-VLAN](07-routage-intervlan.png)

### Vérification du routage inter-VLAN
![Vérification du routage inter-VLAN](08-verification-routage.png)

## 3. DHCP

### Ajout / préparation du DHCP
![DHCP](09-dhcp-add.png)

### Configuration DHCP
![Configuration DHCP](10-dhcp-configuration.png)

### Vérification DHCP
![Vérification DHCP](11-dhcp-verification.png)

### Vérification des baux DHCP
![Baux DHCP](12-dhcp-baux.png)

## 4. Sécurité — ACL-SERVEUR

### Configuration et application de l'ACL
![Configuration ACL-SERVEUR](13-acl-configuration.png)

### PC0 — VLAN 20 : accès au serveur bloqué
![PC0 accès bloqué](14-acl-pc0-bloque.png)

### PC4 — VLAN 10 : accès au serveur autorisé
![PC4 accès autorisé](15-acl-pc4-autorise.png)

## Résultat attendu

- **VLAN 10 — INFORMATIQUE → SRV01 : autorisé**
- **VLAN 20 — EMPLOYES → SRV01 : bloqué**

Ces captures complètent la documentation technique du projet et permettent de vérifier visuellement les différentes étapes de configuration.