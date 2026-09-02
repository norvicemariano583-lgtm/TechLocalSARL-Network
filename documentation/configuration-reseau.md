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
|------|------------|------|
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

## 6. Configuration de l'ACL

Une ACL étendue nommée `ACL-SERVEUR` a été mise en place sur R1 afin de contrôler l'accès au serveur `SRV01` (`192.168.30.10`).

Les règles appliquées sont :

```cisco
ip access-list extended ACL-SERVEUR
 permit ip 192.168.10.0 0.0.0.255 host 192.168.30.10
 deny ip 192.168.20.0 0.0.0.255 host 192.168.30.10
 permit ip any any
```

L'ACL est appliquée en sortie sur la sous-interface du VLAN 30 :

```cisco
interface GigabitEthernet0/0/0.30
 ip access-group ACL-SERVEUR out
```

Cette configuration permet au VLAN 10 (INFORMATIQUE) d'accéder au serveur tout en bloquant l'accès au serveur depuis le VLAN 20 (EMPLOYES). Le reste du trafic est autorisé grâce à la règle `permit ip any any`.

### Vérification de l'application de l'ACL

La commande suivante confirme que l'ACL est bien appliquée en sortie sur `G0/0/0.30` :

```cisco
show ip interface GigabitEthernet0/0/0.30
```

Résultat observé :

```text
Outgoing access list is ACL-SERVEUR
Inbound  access list is not set
```

### Tests de sécurité

Avant l'application de l'ACL, PC4 (VLAN 10) et PC0 (VLAN 20) pouvaient tous les deux joindre `192.168.30.10`.

Après l'application de l'ACL :

| Source | Destination | Résultat |
|--------|-------------|----------|
| PC4 – VLAN 10 | 192.168.30.10 | Autorisé – 0 % de perte |
| PC0 – VLAN 20 | 192.168.30.10 | Bloqué – 100 % de perte |

La commande `show access-lists ACL-SERVEUR` a également confirmé l'utilisation des règles :

```text
permit ip 192.168.10.0 0.0.0.255 host 192.168.30.10 (8 match(es))
deny ip 192.168.20.0 0.0.0.255 host 192.168.30.10 (4 match(es))
permit ip any any
```

La configuration de R1 a ensuite été sauvegardée avec `copy running-config startup-config`.

---

## 7. Prochaine étape

La configuration réseau et le filtrage de l'accès au serveur sont maintenant fonctionnels. Les prochaines améliorations pourront porter sur la documentation des preuves de tests, puis sur d'éventuels services supplémentaires du serveur (DNS, HTTP, etc.) selon les objectifs du projet.
