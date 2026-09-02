# TechLocalSARL – Projet de réseau d'entreprise

## 1. Présentation du projet

Ce projet a été réalisé sur Cisco Packet Tracer dans le cadre de mon apprentissage de l'administration réseau. Il consiste à concevoir et configurer le réseau informatique d'une petite entreprise fictive, TechLocalSARL.

À travers ce projet, je voulais mettre en pratique des notions fondamentales des réseaux d'entreprise : la segmentation en VLAN, le routage inter-VLAN, l'attribution automatique d'adresses via DHCP, et le filtrage du trafic avec des ACL.

## 2. Contexte de l'entreprise

TechLocalSARL est organisée autour de deux services : le service informatique et le service des employés. L'entreprise dispose également d'un serveur central contenant des ressources internes sensibles.

Le réseau doit permettre à chaque utilisateur de travailler normalement, tout en garantissant que l'accès au serveur reste strictement contrôlé.

## 3. Problématique

La question centrale de ce projet est la suivante :

> Comment permettre au service informatique d'accéder au serveur, tout en empêchant les autres employés d'y accéder ?

Pour y répondre, le réseau doit respecter plusieurs contraintes :

- les ordinateurs du service informatique doivent pouvoir communiquer avec le serveur ;
- les ordinateurs des employés ne doivent pas pouvoir accéder au serveur ;
- les deux services doivent être séparés dans des réseaux distincts ;
- le serveur doit être isolé dans un réseau qui lui est propre ;
- les communications entre les réseaux doivent être contrôlées selon les règles de sécurité définies.

## 4. Objectifs du projet

Pour répondre à cette problématique, je vais mettre en place les éléments suivants :

- un VLAN dédié au service informatique ;
- un VLAN dédié aux employés ;
- un VLAN dédié au serveur ;
- un routage inter-VLAN pour permettre la communication entre les réseaux autorisés ;
- un service DHCP pour l'attribution automatique des adresses IP ;
- une ACL pour restreindre l'accès au serveur.

Une fois la configuration terminée, plusieurs tests de connectivité viendront vérifier que le réseau fonctionne comme prévu et que les règles de sécurité sont bien respectées.

## 5. Topologie du réseau

J'ai volontairement choisi une topologie simple, pour progresser étape par étape dans la compréhension du fonctionnement du réseau. Elle comprend :

- 1 routeur ;
- 1 switch ;
- plusieurs ordinateurs ;
- 1 serveur ;
- des imprimantes réparties entre les différents services.

![Topologie du réseau](topology.png)

## 6. Organisation des VLAN

Trois VLAN sont utilisés pour séparer logiquement les différents éléments du réseau au sein du même switch :

| VLAN | Nom | Utilisation |
|------|-----|-------------|
| 10 | INFORMATIQUE | Ordinateurs du service informatique |
| 20 | EMPLOYES | Ordinateurs des employés |
| 30 | SERVEURS | Serveur de l'entreprise |

## 7. Plan d'adressage IP

Chaque VLAN dispose de son propre réseau IP, avec une passerelle dédiée :

| VLAN | Réseau | Passerelle |
|------|--------|------------|
| VLAN 10 | 192.168.10.0/24 | 192.168.10.1 |
| VLAN 20 | 192.168.20.0/24 | 192.168.20.1 |
| VLAN 30 | 192.168.30.0/24 | 192.168.30.1 |

Le serveur, quant à lui, conserve une adresse IP fixe : **192.168.30.10**
