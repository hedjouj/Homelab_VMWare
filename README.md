# Homelab Active Directory avec Windows Server 2019

## Introduction
Ce projet consiste à créer un environnement Active Directory complet en virtualisation pour apprendre l'administration système. J'ai construit une infrastructure réseau avec un contrôleur de domaine et des clients Windows 10, en utilisant VMware Workstation.
L'objectif est de comprendre comment fonctionnent les services d'annuaire dans un environnement professionnel, notamment la gestion centralisée des utilisateurs, des groupes et des permissions.
Technologies utilisées

VMware Workstation Player (virtualisation)
Windows Server 2019 Standard Evaluation
Windows 10 Professionnel
Active Directory Domain Services
DNS (Domain Name System)
DHCP (Dynamic Host Configuration Protocol)

Architecture réseau
L'infrastructure utilise une architecture à deux réseaux :
Serveur DC (Domain Controller)

Carte 1 (INTERNET) : Réseau NAT (VMnet8) pour l'accès Internet
Carte 2 (INTERNE) : Réseau Host-only (VMnet1, subnet 172.16.0.0/24) pour le réseau local

Client Windows 10

Carte unique : Réseau Host-only (VMnet1) connecté au réseau interne uniquement

Cette configuration permet d'isoler les clients sur un réseau privé sécurisé, tandis que le serveur peut accéder à Internet pour les mises à jour.
