Voici une vue claire et complète du **modèle OSI (Open Systems Interconnection)** — une manière standard de décomposer les communications réseau en **7 couches** pour mieux concevoir, diagnostiquer et faire dialoguer des systèmes hétérogènes. C’est un **modèle conceptuel** (le vrai Internet suit plutôt TCP/IP), mais il reste la meilleure « carte mentale » pour raisonner réseau.

# Vue d’ensemble

* **But :** standardiser les rôles du réseau, découper les responsabilités et faciliter l’interopérabilité.
* **Idée clé :** à l’émission, chaque couche **encapsule** les données de la couche au-dessus (ajoute son en-tête/étiquette) ; à la réception, on **décapsule** dans l’ordre inverse.
* **Du haut vers le bas :** 7 Application → 6 Présentation → 5 Session → 4 Transport → 3 Réseau → 2 Liaison de données → 1 Physique.

## Les 7 couches, de haut en bas

### 7 — Application

* **Rôle :** point de contact avec l’application (navigateur, client mail, API…). Définit les **protocoles applicatifs** et leurs conventions.
* **Exemples :** HTTP/1.1–2–3, DNS, SMTP/IMAP/POP, FTP/SFTP, DHCP (côté « logique d’app »), SNMP, SIP.
* **Remarque :** dans la pratique Internet, beaucoup de ce qu’OSI place en « Présentation/Session » est inclus ici.

### 6 — Présentation

* **Rôle :** **format** des données et **sécurité** : encodage, sérialisation, chiffrement, compression.
* **Exemples :** TLS/SSL (souvent rangé ici ou en Session selon les écoles), X.509, ASN.1, JSON, XML, JPEG/MP3 (côté encodage), gzip.
* **Fonctions :** chiffrement/déchiffrement, (dé)compression, conversion de formats/jeux de caractères.

### 5 — Session

* **Rôle :** **dialogue** entre applications : ouverture/fermeture de session, rétablissement après coupure, points de reprise.
* **Exemples :** RPC, NetBIOS session, gestion de session TLS (selon interprétation), contrôles de dialogue dans certains middlewares.

### 4 — Transport

* **Rôle :** **fiabilité, flux, segmentation, ports**. Transporte de bout en bout entre processus applicatifs.
* **Adressage :** **ports** (TCP/UDP).
* **Unités :** **segments** (TCP) / **datagrammes** (UDP).
* **Exemples :**

  * **TCP** : fiable (ACK, fenêtres, retransmission), ordonné, contrôle de congestion.
  * **UDP** : non fiable, sans connexion, faible latence.
  * **QUIC** : transport moderne chiffré, au-dessus d’UDP, fournit fiabilité + multiplexage (on le classe **Transport**).

### 3 — Réseau

* **Rôle :** **routage** entre réseaux, **adressage logique**.
* **Adressage :** **IP** (IPv4/IPv6).
* **Unité :** **paquets**.
* **Exemples :** IP, ICMP, **routage** (OSPF, EIGRP, IS-IS, BGP), **NAT** (agit surtout sur L3/L4), IPsec (mode tunnel/transport, sécurité L3).

### 2 — Liaison de données

* **Rôle :** communication **intra-lien** (même segment), **adressage MAC**, détection d’erreur **FCS**, contrôle d’accès au média.
* **Adressage :** **MAC** (Ethernet).
* **Unité :** **trames**.
* **Exemples :** Ethernet (802.3), VLAN 802.1Q, STP/RSTP, LACP, PPP, Wi-Fi 802.11, ARP\*.

  * *ARP relie L2↔L3 (très souvent rangé « 2.5 » : résolution IP→MAC).*
* **Équipements :** **commutateurs (switches)**, **ponts** ; **MPLS** est parfois dit « 2.5 ».

### 1 — Physique

* **Rôle :** **bits** sur le médium : électrique, optique, radio. Connecteurs, câbles, modulation, synchronisation.
* **Unité :** **bits**.
* **Exemples :** cuivre (RJ45/Cat5e–Cat6), fibre (SM/MM), radio Wi-Fi (modulation OFDM), 1000BASE-T, 10GBASE-SR, Bluetooth, NRZ/PAM-4.
* **Équipements :** **répéteurs**, **hubs**, **transceivers** (SFP/GBIC).

---

## Résumé express (PDUs, adresses, exemples)

| Couche         | Unité (PDU)        | Adressage | Exemples                              | Équipements typiques       |
| -------------- | ------------------ | --------- | ------------------------------------- | -------------------------- |
| 7 Application  | Données            | —         | HTTP, DNS, SMTP, REST/JSON            | —                          |
| 6 Présentation | Données            | —         | TLS, X.509, ASN.1, gzip               | —                          |
| 5 Session      | Données            | —         | RPC, gestion de session               | —                          |
| 4 Transport    | Segment/Datagramme | **Port**  | TCP, UDP, QUIC                        | pare-feu L4                |
| 3 Réseau       | Paquet             | **IP**    | IPv4/IPv6, ICMP, OSPF/BGP, IPsec, NAT | **routeurs**, firewalls L3 |
| 2 Liaison      | Trame              | **MAC**   | Ethernet, 802.1Q, STP, Wi-Fi, ARP     | **switches**, AP Wi-Fi     |
| 1 Physique     | Bits               | —         | 1000BASE-T, fibre, radio              | répéteurs, câbles, SFP     |

---

## Encapsulation (aller) & décapsulation (retour)

Exemple : un navigateur envoie `GET /` vers un site HTTPS.

1. **L7–L5** : l’appli forme la requête HTTP. TLS peut chiffrer (handshake, clés).
2. **L4** : **TCP** segmente, pose **ports** (client éphémère → 443), ajoute fiabilité.
3. **L3** : **IP** source/destination, choix du **routage**.
4. **L2** : **MAC** source/destination locales (via **ARP**), **FCS** d’erreur, éventuel **VLAN**.
5. **L1** : transformation en **bits** sur le lien.

À l’arrivée, chaque couche supprime son en-tête et remet les données à la couche supérieure.

---

## Où placer… (questions fréquentes)

* **TLS/SSL :** OSI le situe en **Présentation (6)** / parfois **Session (5)** ; dans l’Internet réel, c’est « au-dessus de Transport », donc pratiquement côté **application** de l’ingénieur réseau.
* **QUIC/HTTP/3 :** **QUIC = Transport (4)** sur **UDP** ; **HTTP/3 = Application (7)**.
* **VPN :** **IPsec** (L3), **SSL VPN** (TLS, donc « haut »), **MPLS** (entre L2 et L3).
* **NAT** : modifie champs **IP** (L3) et parfois **ports** (PAT, L4).
* **VLAN** : **L2** (802.1Q). **Trunk** = lien L2 transportant plusieurs VLAN.

---

## Appareils et couches (raccourci utile)

* **Hub / répéteur :** L1.
* **Switch (Ethernet) :** L2 (table MAC, VLAN, STP).
* **Switch L3 / Routeur :** L3 (routage IP, sous-réseaux).
* **Pare-feu traditionnel :** L3/L4 (IP/ports). **NGFW** : jusqu’à L7 (inspection applicative).
* **Load balancer / proxy :** L4–L7 (NAT, TLS offload, HTTP rewriting).

---

## Contrôle d’erreurs & de flux : qui fait quoi ?

* **L1/L2 :** détection d’erreurs locales (FCS), retransmissions possibles sur certains liens (Wi-Fi).
* **L3 :** ICMP (diagnostic), fragmentation IP (éviter si possible ; PMTUD).
* **L4 :** fiabilité (TCP), contrôle de congestion (CUBIC/BBR), **fenêtrage**.
* **L7 :** réessais applicatifs (HTTP, clients, middlewares).

---

## OSI vs. TCP/IP (modèle Internet)

| OSI            | TCP/IP (pratique Internet)                              |
| -------------- | ------------------------------------------------------- |
| 7 Application  | **Application** (inclut souvent Présentation + Session) |
| 6 Présentation |                                                         |
| 5 Session      |                                                         |
| 4 Transport    | **Transport** (TCP, UDP, QUIC)                          |
| 3 Réseau       | **Internet** (IP, ICMP, routage)                        |
| 2 Liaison      | **Accès réseau** (Ethernet, Wi-Fi, ARP…)                |
| 1 Physique     |                                                         |

---

## Méthode de dépannage « en couches »

1. **L1 — Physique :** lien up ? câbles/optique/SFP ok ? LED, `ethtool`.
2. **L2 — Liaison :** MAC vue ? VLAN correct ? STP/port-security ? `show mac`, `bridge fdb`.
3. **L3 — Réseau :** IP/gateway, routes, `ping`, `traceroute`, `show ip route`, ACL/NAT.
4. **L4 — Transport :** ports ouverts ? **TCP SYN/ACK** ? `telnet host port` / `nc`, pare-feu L4.
5. **L5–L7 — Session/Prés./App :** TLS (certificats, SNI), HTTP (codes 4xx/5xx), DNS ok ?

---

## Mnémoniques (FR & EN)

* **Descendant (7→1)** : *«**A**pplication **P**résentation **S**ession **T**ransport **R**éseau **L**iaison **P**hysique»*
  (EN : *All People Seem To Need Data Processing*)
* **Montant (1→7)** : *«**P**hysique **L**iaison **R**éseau **T**ransport **S**ession **P**résentation **A**pplication»*

---

### À retenir

* OSI est **pédagogique** : il structure votre pensée et votre **méthode de triage**.
* Les **adresses** changent selon la couche : **MAC (L2)**, **IP (L3)**, **Port (L4)**.
* Les **unités** changent aussi : **Bits → Trames → Paquets → Segments → Données**.
* Dans la vraie vie, beaucoup d’éléments « Présentation/Session » sont intégrés aux **applications** ou aux **transports modernes** (TLS, QUIC).

Si tu veux, je peux te préparer une **fiche A4** récapitulant tout (tableaux, schéma d’encapsulation) prête à imprimer.
