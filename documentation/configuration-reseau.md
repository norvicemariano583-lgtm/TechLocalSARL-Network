# Configuration du réseau

## 1. Création des VLAN

Pour séparer les différents services de l'entreprise, trois VLAN ont été créés sur le switch SW1.

| VLAN | Nom | Utilisation |
|------|-----|-------------|
| 10 | INFORMATIQUE | Service informatique |
| 20 | EMPLOYES | Employés |
| 30 | SERVEURS | Serveur interne |

Les ports du switch ont ensuite été affectés au VLAN correspondant à chaque service.

---

## 2. Affectation des ports

| Port | Équipement | VLAN |
|------|-------------|------|
| Fa0/1 | PC0 | 20 |
| Fa0/2 | PC1 | 20 |
| Fa0/3 | PC2 | 20 |
| Fa0/4 | Printer0 | 20 |
| Fa0/5 | PC3 | 20 |
| Fa0/6 | PC4 | 10 |
| Fa0/7 | PC5 | 10 |
| Fa0/8 | Printer1 | 10 |
| Fa0/24 | SRV01 | 30 |
| Gi0/1 | R1 | Trunk |

---

## 3. Trunk entre SW1 et R1

Le port GigabitEthernet0/1 de SW1 est configuré en mode trunk.

Ce trunk permet de transporter les VLAN 10, 20 et 30 entre le switch et le routeur R1.

L'encapsulation utilisée est 802.1Q.

---

## 4. Routage inter-VLAN

Le routage inter-VLAN est réalisé sur le routeur R1 avec des sous-interfaces.

| Sous-interface | VLAN | Adresse IP |
|----------------|------|------------|
| G0/0/0.10 | 10 | 192.168.10.1 |
| G0/0/0.20 | 20 | 192.168.20.1 |
| G0/0/0.30 | 30 | 192.168.30.1 |

Ces adresses servent de passerelles par défaut pour les équipements de chaque réseau.

---

## 5. Configuration DHCP

Le service DHCP est configuré directement sur R1.

### VLAN 10 – INFORMATIQUE

Réseau :

`192.168.10.0/24`

Passerelle :

`192.168.10.1`

Les adresses de `192.168.10.1` à `192.168.10.20` sont exclues du pool DHCP.

### VLAN 20 – EMPLOYES

Réseau :

`192.168.20.0/24`

Passerelle :

`192.168.20.1`

Les adresses de `192.168.20.1` à `192.168.20.20` sont exclues du pool DHCP.

### Vérification

La commande `show ip dhcp binding` permet de vérifier les adresses actuellement attribuées.

Au moment de cette vérification, 8 baux DHCP étaient présents :

- 3 adresses dans le VLAN 10 ;
- 5 adresses dans le VLAN 20.

---

## 6. Prochaine étape

La prochaine étape du projet sera de configurer le serveur SRV01 avec une adresse IP statique, puis de réaliser les tests de connectivité.

Une ACL sera ensuite mise en place afin d'autoriser le service informatique à accéder au serveur tout en bloquant l'accès depuis le VLAN des employés.
