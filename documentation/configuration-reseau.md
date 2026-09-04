# Configuration du réseau

## 1. Création des VLAN

Trois VLAN ont été créés sur SW1 afin de séparer les services de l'entreprise.

| VLAN | Nom | Utilisation |
|---|---|---|
| 10 | INFORMATIQUE | Service informatique |
| 20 | EMPLOYES | Employés |
| 30 | SERVEURS | Serveur interne |

## 2. Affectation des ports

| Port | Équipement | VLAN |
|---|---|---|
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

## 3. Trunk et routage inter-VLAN

SW1 et R1 sont reliés par un trunk 802.1Q sur `Gi0/1`. Le routage inter-VLAN est assuré par les sous-interfaces de R1.

| Sous-interface | VLAN | Adresse IP |
|---|---:|---|
| G0/0/0.10 | 10 | 192.168.10.1/24 |
| G0/0/0.20 | 20 | 192.168.20.1/24 |
| G0/0/0.30 | 30 | 192.168.30.1/24 |

## 4. DHCP

Le DHCP est fourni par R1.

- VLAN 10 : `192.168.10.0/24`, passerelle `192.168.10.1`, DNS `192.168.30.10`.
- VLAN 20 : `192.168.20.0/24`, passerelle `192.168.20.1`, DNS `192.168.30.10`.
- Les adresses `.1` à `.20` de chaque réseau sont exclues du DHCP.

## 5. DNS

`SRV01` possède l'adresse fixe `192.168.30.10`.

Le service DNS est activé avec l'enregistrement :

| Nom | Type | Adresse |
|---|---|---|
| `srv01.techlocal.local` | A | `192.168.30.10` |

Les clients reçoivent l'adresse du DNS via DHCP.

## 6. ACL-SERVEUR

Une ACL étendue contrôle l'accès au serveur. Le VLAN 20 peut utiliser le DNS mais ne peut pas accéder directement au serveur pour les autres communications. Le VLAN 10 conserve l'accès au serveur.

```cisco
ip access-list extended ACL-SERVEUR
 permit udp 192.168.20.0 0.0.0.255 host 192.168.30.10 eq domain
 permit tcp 192.168.20.0 0.0.0.255 host 192.168.30.10 eq domain
 deny ip 192.168.20.0 0.0.0.255 host 192.168.30.10
 permit ip 192.168.10.0 0.0.0.255 host 192.168.30.10
 permit ip any any
```

Application :

```cisco
interface GigabitEthernet0/0/0.30
 ip access-group ACL-SERVEUR out
```

## 7. Vérifications et tests

### ACL

```cisco
show ip interface GigabitEthernet0/0/0.30
show access-lists ACL-SERVEUR
```

`G0/0/0.30` confirme que `ACL-SERVEUR` est appliquée en sortie.

### PC0 – VLAN 20

- `nslookup srv01.techlocal.local` : résolution réussie vers `192.168.30.10`.
- `ping 192.168.30.10` : bloqué, 100 % de perte.

### PC4 – VLAN 10

- `ping 192.168.30.10` : réussi, 0 % de perte.
- `ping srv01.techlocal.local` : résolution et communication réussies, 0 % de perte.

## 8. Sécurisation de R1

### Mode privilégié

Un `enable secret` est configuré. `service password-encryption` est également activé.

### Console

La console est protégée par authentification et déconnexion automatique après 10 minutes d'inactivité :

```cisco
line console 0
 password <mot-de-passe-console>
 login
 logging synchronous
 exec-timeout 10 0
```

### Accès VTY et SSH

Les lignes VTY utilisent la base d'utilisateurs locale et acceptent uniquement SSH :

```cisco
username admin privilege 15 secret <secret-admin>

line vty 0 4
 login local
 transport input ssh
 exec-timeout 10 0
```

SSH est activé en version 2 :

```text
SSH Enabled - version 2.0
```

Un nom de domaine est configuré et des clés RSA sont utilisées pour SSH. Le compte `admin` possède le niveau de privilège 15.

> **Sécurité :** les mots de passe et secrets réels ne sont pas publiés dans ce dépôt GitHub.

### Administration SSH locale

L'infrastructure reste entièrement locale. SSH permet néanmoins à un poste du LAN d'administrer R1 via son adresse IP de passerelle, sans connexion physique à la console. Telnet n'est pas autorisé.

## 9. Sauvegarde de la configuration

Après chaque modification importante de R1, la configuration doit être enregistrée :

```cisco
copy running-config startup-config
```

## 10. État du projet

À ce stade, le projet comprend :

- segmentation VLAN ;
- trunk 802.1Q ;
- routage inter-VLAN ;
- DHCP ;
- DNS ;
- ACL de contrôle d'accès au serveur ;
- sécurisation des accès administratifs de R1 ;
- administration SSH locale ;
- tests de connectivité, DNS et sécurité validés dans Cisco Packet Tracer.

Les captures sont regroupées dans [`screenshots/`](../screenshots/README.md).