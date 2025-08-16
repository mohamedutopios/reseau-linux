Voici un **programme de formation Linux Réseau sur 3 jours** structuré, progressif et orienté pratique, adapté pour des apprenants souhaitant maîtriser la configuration réseau sous Linux dans un contexte professionnel.

---

## **📅 Jour 1 – Fondamentaux du réseau sous Linux**

### **Matin : Introduction et bases**

* **Présentation des concepts réseaux**

  * Modèle OSI & TCP/IP
  * Adressage IPv4 et IPv6
  * Notion de passerelle, masque, DNS
* **Commandes de base réseau**

  * `ip addr`, `ip link`, `ip route`
  * `ping`, `traceroute`, `nslookup`, `dig`
* **Configuration IP statique et DHCP**

  * Édition manuelle de la configuration (`/etc/network/interfaces`, NetworkManager)
  * Activation/désactivation des interfaces

**TP** :

* Afficher la configuration réseau d’une VM Linux
* Passer d’une configuration DHCP à une configuration IP statique

---

### **Après-midi : Exploration réseau et outils de diagnostic**

* **Analyse des connexions**

  * `netstat`, `ss`, `lsof`
* **Vérification de la connectivité**

  * ICMP, ports ouverts, état des sockets
* **Résolution de noms**

  * Fonctionnement du DNS
  * Modification de `/etc/hosts` et `/etc/resolv.conf`
* **Introduction au pare-feu**

  * Présentation d’`iptables` et `ufw`

**TP** :

* Identifier les ports ouverts sur la machine
* Configurer un DNS manuel et tester la résolution

---

## **📅 Jour 2 – Services réseaux et routage**

### **Matin : Routage et NAT**

* **Table de routage**

  * Ajouter/supprimer des routes
  * Routes statiques vs dynamiques
* **NAT et masquerading**

  * Fonctionnement et configuration avec `iptables`
* **Passerelle Linux**

  * Activer le routage IP (`/proc/sys/net/ipv4/ip_forward`)

**TP** :

* Mettre en place une machine Linux comme routeur entre deux réseaux
* Tester le trafic entre deux sous-réseaux via NAT

---

### **Après-midi : Services réseau essentiels**

* **Serveur SSH**

  * Installation et configuration (`sshd_config`)
  * Gestion des clés et sécurité
* **Serveur DHCP**

  * Installation, configuration de plages d’adresses
* **Serveur DNS**

  * Installation et configuration de `bind9` ou `dnsmasq`

**TP** :

* Installer et configurer un serveur SSH avec authentification par clé
* Configurer un serveur DHCP distribuant des IP dans un sous-réseau test
* Mettre en place un DNS local pour un domaine fictif

---

## **📅 Jour 3 – Sécurité, surveillance et dépannage**

### **Matin : Sécurité réseau**

* **Pare-feu avancé**

  * Règles `iptables` filtrage entrant/sortant
  * Utilisation de `ufw` ou `firewalld`
* **Protection SSH**

  * Fail2Ban
  * Restriction d’accès par IP
* **Audit réseau**

  * `nmap` : découverte et analyse des services

**TP** :

* Configurer un pare-feu pour n’autoriser que SSH et HTTP
* Mettre en place Fail2Ban contre les attaques par force brute

---

### **Après-midi : Supervision et dépannage**

* **Supervision réseau**

  * `iftop`, `nload`, `vnstat` pour surveiller le trafic
  * Analyse avec `tcpdump` et Wireshark
* **Journalisation**

  * Analyse des logs réseau (`/var/log/syslog`, `/var/log/auth.log`)
* **Dépannage**

  * Cas pratiques : panne réseau, DNS, passerelle, firewall

**TP final** :

* Diagnostiquer et corriger une panne réseau simulée
* Mettre en place un mini-lab avec 3 machines :

  * Un routeur Linux
  * Un serveur (SSH, HTTP)
  * Un poste client

---

