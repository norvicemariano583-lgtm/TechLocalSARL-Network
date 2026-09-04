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

Réseau : `192.168.10.0/24`

Passerelle : `192.168.10.1`

DNS distribué : `192.168.30.10`

Les adresses de `192.168.10.1` à `192.168.10.20` sont exclues du pool DHCP.

### VLAN 20 – EMPLOYES

Réseau : `192.168.20.0/24`

Passerelle : `192.168.20.1`

DNS distribué : `192.168.30.10`

Les adresses de `192.168.20.1` à `192.168.20.20` sont exclues du pool DHCP.

### Vérification

La commande `show ip dhcp binding` permet de vérifier les adresses actuellement attribuées.

Les postes clients utilisent DHCP pour obtenir automatiquement leur adresse IP, leur passerelle et l'adresse du serveur DNS.

---

## 6. Service DNS sur SRV01

Le serveur `SRV01` utilise l'adresse IP fixe `192.168.30.10`.

Le service DNS est activé sur le serveur et un enregistrement de type A a été créé :

| Nom | Type | Adresse |
|-----|------|---------|
| `srv01.techlocal.local` | A | `192.168.30.10` |

Les clients reçoivent `192.168.30.10` comme serveur DNS via DHCP.

---

## 7. Configuration de l'ACL

Une ACL étendue nommée `ACL-SERVEUR` a été mise en place sur R1 afin de contrôler l'accès au serveur `SRV01` (`192.168.30.10`).

La configuration finale prend en compte le fonctionnement du DNS : le VLAN 20 peut interroger le serveur DNS, mais reste interdit d'accès direct au serveur pour les autres communications.

```cisco
ip access-list extended ACL-SERVEUR
 permit udp 192.168.20.0 0.0.0.255 host 192.168.30.10 eq domain
 permit tcp 192.168.20.0 0.0.0.255 host 192.168.30.10 eq domain
 deny ip 192.168.20.0 0.0.0.255 host 192.168.30.10
 permit ip 192.168.10.0 0.0.0.255 host 192.168.30.10
 permit ip any any
```

L'ACL est appliquée en sortie sur la sous-interface du VLAN 30 :

```cisco
interface GigabitEthernet0/0/0.30
 ip access-group ACL-SERVEUR out
```

Cette configuration permet :

- au VLAN 10 (INFORMATIQUE) d'accéder au serveur ;
- au VLAN 20 (EMPLOYES) d'utiliser le DNS du serveur ;
- au VLAN 20 de ne pas accéder directement aux ressources du serveur ;
- au reste du trafic de circuler conformément à la règle `permit ip any any`.

---

## 8. Vérification de l'application de l'ACL

La commande suivante confirme que l'ACL est bien appliquée en sortie sur `G0/0/0.30` :

```cisco
show ip interface GigabitEthernet0/0/0.30
```

Résultat observé :

```text
Outgoing access list is ACL-SERVEUR
Inbound access list is not set
```

La commande suivante permet également de contrôler les compteurs des différentes règles :

```cisco
show access-lists ACL-SERVEUR
```

---

## 9. Tests de sécurité et de résolution DNS

### PC0 – VLAN 20 EMPLOYES

Le test de connectivité directe vers le serveur est bloqué :

```text
ping 192.168.30.10
```

Résultat : `100 %` de perte.

La résolution DNS fonctionne cependant :

```text
nslookup srv01.techlocal.local
```

Le serveur DNS `192.168.30.10` répond et retourne :

```text
Name:    srv01.techlocal.local
Address: 192.168.30.10
```

Le test démontre donc que le VLAN 20 peut utiliser le service DNS sans obtenir un accès direct au serveur.

### PC4 – VLAN 10 INFORMATIQUE

Le poste informatique peut joindre directement le serveur :

```text
ping 192.168.30.10
```

Résultat : `0 %` de perte.

La résolution du nom fonctionne également et le ping suivant aboutit :

```text
ping srv01.techlocal.local
```

Résultat : `0 %` de perte.

### Synthèse

| Source | Destination | Résultat |
|--------|-------------|----------|
| PC0 – VLAN 20 | DNS `192.168.30.10` | Autorisé |
| PC0 – VLAN 20 | Serveur `192.168.30.10` en ICMP | Bloqué – 100 % de perte |
| PC4 – VLAN 10 | Serveur `192.168.30.10` en ICMP | Autorisé – 0 % de perte |
| PC4 – VLAN 10 | `srv01.techlocal.local` | Autorisé – 0 % de perte |

Ces tests valident le fonctionnement de la politique de filtrage mise en place par `ACL-SERVEUR`.

---

## 10. Preuves visuelles

Les captures de configuration et de tests sont regroupées dans le dossier [`screenshots/`](../screenshots/README.md) et classées chronologiquement.

Elles documentent notamment :

- la création et l'affectation des VLAN ;
- la configuration et la vérification du trunk ;
- le routage inter-VLAN ;
- la configuration et les baux DHCP ;
- la configuration de `ACL-SERVEUR` ;
- le blocage de PC0 ;
- l'autorisation de PC4.

---

## 11. État du projet

La segmentation VLAN, le trunk, le routage inter-VLAN, le DHCP, le DNS et le filtrage ACL sont configurés et vérifiés dans Cisco Packet Tracer.

La configuration finale doit être sauvegardée dans la mémoire de démarrage de R1 avec :

```cisco
copy running-config startup-config
```
