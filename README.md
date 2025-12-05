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

![Configuration VMnet1 dans Virtual Network Editor](Screenshot/)

## Étape 1 : Installation de VMware Workstation
J'ai téléchargé VMware Workstation Pro depuis le site officiel de VMware. L'installation est simple et ne nécessite pas de licence pour un usage personnel et non commercial.

## Étape 2 : Configuration du Virtual Network Editor
Avant de créer les machines virtuelles, j'ai configuré les réseaux virtuels dans VMware pour correspondre à l'architecture souhaitée.
Dans le menu Edit, j'ai ouvert le Virtual Network Editor avec les droits administrateur. J'ai vérifié la présence de VMnet1 (Host-only) et configuré les paramètres suivants :

Type : Host-only
Subnet IP : 172.16.0.0
Masque : 255.255.255.0
DHCP : Désactivé (important car le serveur DC sera le serveur DHCP)
Adaptateur hôte : Décoché (pour éviter les conflits d'IP)

![Configuration VMnet1 dans Virtual Network Editor](Screenshot/config-vmnet1.png)

## Étape 3 : Installation de Windows Server 2019
J'ai créé une nouvelle machine virtuelle avec les spécifications suivantes :

Nom : DC-Server
Type : Windows Server 2019
RAM : 4 Go
Disque : 60 Go
Processeurs : 1 processeur, 2 cœurs
Firmware : BIOS (meilleure compatibilité)

Pour l'installation, j'ai téléchargé l'ISO de Windows Server 2019 Standard Evaluation (180 jours gratuits) depuis le site Microsoft Evaluation Center. Il est important de choisir la version "Desktop Experience" pour avoir l'interface graphique.
Pendant l'installation, j'ai sélectionné "Custom: Install Windows only (advanced)" et laissé Windows créer automatiquement les partitions sur le disque.
Le mot de passe administrateur défini est Password1! (à changer en production réelle).
![](Screenshot/install-Windows_server.png)
![](Screenshot/DC_home.png)

## Étape 4 : Configuration réseau du serveur
Une fois Windows Server installé, j'ai configuré les deux cartes réseau.
Pour y accéder rapidement, j'ai utilisé la commande Windows + R puis tapé ncpa.cpl.
J'ai renommé les cartes pour plus de clarté :

Ethernet0 renommé en INTERNET
Ethernet1 renommé en INTERNE

Configuration carte INTERNET (NAT)

Laisser en DHCP automatique
Cette carte reçoit une IP de VMware (192.168.x.x)
Elle permet au serveur d'accéder à Internet

Configuration carte INTERNE (Host-only)

IP : 172.16.0.1
Masque : 255.255.255.0
Passerelle : Vide (le serveur est lui-même la passerelle)
DNS : 127.0.0.1 (le serveur est son propre DNS)

J'ai vérifié la configuration avec la commande ipconfig /all dans l'invite de commandes.
![](Screenshot/Ipconfig_result.png)

## Étape 5 : Installation d'Active Directory Domain Services
Active Directory est le service qui permet de gérer les utilisateurs, les ordinateurs et les ressources de manière centralisée dans le domaine.
Dans Server Manager, j'ai cliqué sur "Add roles and features" et sélectionné :

Installation type : Role-based
Server : Mon serveur (WIN-XXXXXXX)
Server Roles : Active Directory Domain Services
J'ai ajouté les features requises quand demandé
Lancé l'installation

Après l'installation du rôle, j'ai cliqué sur le drapeau jaune en haut à droite de Server Manager pour promouvoir le serveur en contrôleur de domaine.
Configuration du domaine

Déploiement : Ajouter une nouvelle forêt
Nom de domaine racine : monlab.local
Niveau fonctionnel : Windows Server 2016
Services : DNS et Catalogue global cochés
Mot de passe DSRM : Password1! (pour la récupération d'urgence)
Nom NetBIOS : MONLAB (généré automatiquement)

L'installation a pris environ 10 à 15 minutes. Le serveur a redémarré automatiquement.
Après le redémarrage, l'écran de connexion montre maintenant MONLAB\Administrator, confirmant que le domaine est actif.
![](Screenshot/rôles_installés.png)

## Étape 6 : Configuration de DHCP
Le serveur DHCP distribue automatiquement les adresses IP aux clients du réseau.
J'ai installé le rôle DHCP depuis Server Manager en suivant le même processus que pour Active Directory.
Une fois installé, j'ai ouvert le gestionnaire DHCP (Tools > DHCP) et configuré une nouvelle étendue :

Nom : Réseau Interne
Plage d'adresses : 172.16.0.100 à 172.16.0.200 (101 adresses disponibles)
Masque : 255.255.255.0
Durée du bail : 8 jours
Routeur (passerelle) : 172.16.0.1
Serveur DNS : 172.16.0.1

Important : J'ai autorisé le serveur DHCP dans Active Directory en faisant un clic droit sur le nom du serveur dans le gestionnaire DHCP et en sélectionnant "Authorize". La flèche verte indique que le serveur est autorisé et actif.
![](Screenshot/étendue_DHCP.png)
