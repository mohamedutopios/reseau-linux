Parfait — allons au fond des choses. Une **trame Ethernet** (Ethernet II, le format le plus courant) est l’unité que la carte réseau émet/reçoit en **couche 2**. Elle entoure (encapsule) le paquet de la couche 3 (souvent IP). Voici sa composition **exacte**, champ par champ, avec les tailles et le rôle de chacun.

# 1) Structure “sur le fil” (ordre d’émission)

```
Préambule (7) | SFD (1) | MAC Dest (6) | MAC Src (6) | [802.1Q Tag (4) optionnel]
| EtherType/Longueur (2) | Données/Payload (46–1500) | FCS/CRC-32 (4)
```

*(Tailles en octets. Le préambule et le SFD ne sont pas “livrés” à l’OS : ils servent à la synchro physique.)*

---

## A. Préambule — 7 octets (0x55 répétés)

* Bits `10101010` répétés → **synchronisation d’horloge** entre émetteur et récepteur.
* Aide la carte distante à “se caler” sur le rythme des bits.

## B. SFD (Start Frame Delimiter) — 1 octet (0xD5)

* Marque le **début réel** de la trame (après la synchro).
* Valeur binaire `10101011` (diffère du préambule par son dernier bit).

> 🧠 **Préambule + SFD** sont gérés par la **couche 1**. La pile réseau de l’OS ne les voit pas.

---

## C. En-tête Ethernet de base — 14 octets

### 1) Adresse MAC destination — 6 octets

* **48 bits**. Format : `xx:xx:xx:xx:xx:xx`.
* **Bit I/G** (Individual/Group, le LSB du premier octet) :

  * 0 = unicast, 1 = **multicast** (et `FF:FF:...` = **broadcast**).
* **Bit U/L** (Universel/Local, avant-dernier bit du premier octet) :

  * 0 = universel (OUI constructeur), 1 = **localement administré**.
* Sert au **switch L2** pour le **comportement de commutation** (table MAC).

### 2) Adresse MAC source — 6 octets

* MAC de l’émetteur (mêmes codes de bits U/L & I/G, mais I/G doit être 0 ici).

### 3) EtherType (Ethernet II) **ou** Longueur (IEEE 802.3) — 2 octets

* **EtherType** si valeur ≥ `0x0600` (1536 déc.) :

  * `0x0800` = IPv4
  * `0x86DD` = IPv6
  * `0x0806` = ARP
  * `0x8847/0x8848` = MPLS
  * `0x8864` = PPPoE Data
* **Longueur** (≤ 1500) dans les trames **802.3** “historiques” → la suite est alors un en-tête **LLC (DSAP/SSAP/Control)**, éventuellement **SNAP** (OUI + Protocol ID). En pratique, **Ethernet II domine** aujourd’hui.

---

## D. Tag VLAN 802.1Q (optionnel) — 4 octets

Inséré **après MAC Source** et **avant EtherType** :

* **TPID** : 2 octets, généralement `0x8100` (S-Tag `0x88A8` pour QinQ).
* **TCI** (Tag Control Information) : 2 octets décomposés en :

  * **PCP** (3 bits) : priorité (**802.1p**, 0–7).
  * **DEI** (1 bit) : Drop Eligibility Indicator.
  * **VID** (12 bits) : **VLAN ID** (1–4094 ; 0 et 4095 réservés).
* **Effet sur la taille** :

  * **Untagged** : max standard 1518 octets (de MAC dest au FCS).
  * **Tagged 802.1Q** : max **1522** (standard 802.3ac).
  * **Mini** (voir plus bas) : 64 (untagged) / **68** (tagged).

> **QinQ (802.1ad)** : double tag (S-Tag `0x88A8` + C-Tag `0x8100`) → +8 octets.

---

## E. Données / Payload — **46 à 1500 octets**

* Contient **typiquement un paquet L3** (ex. IP) + L4/L7 au-dessus.
* **MTU L3 “classique”** = **1500** pour IP sur Ethernet (hors VLANs et protocoles spéciaux).
* Si le contenu est **< 46 octets**, on ajoute du **padding** (zéros) pour atteindre la taille minimale de trame.
* Exemples fréquents :

  * IPv4/IPv6 (EtherType 0x0800 / 0x86DD)
  * ARP (0x0806) — **ARP fait 28 octets**, donc **padding** nécessaire.
  * PPPoE (0x8863/0x8864) réduit la MTU IP (souvent 1492).

---

## F. FCS (Frame Check Sequence) — 4 octets

* **CRC-32** calculé sur tout (de **MAC Dest** jusqu’à la **fin du payload**).
* **Polynôme** standard Ethernet : `0x04C11DB7`.
* Permet de **détecter des erreurs** (inversions de bits, rafales courtes…).
* Si le CRC reçu ≠ CRC calculé → **trame rejetée** par la NIC (non transmise à l’OS).

---

## G. Interframe Gap (IFG) — 12 octets “de silence” (96 bit-times)

* Temps de repos **entre deux trames** pour laisser respirer le support.
* **Pas** une partie de la trame, mais exigence de la couche 1.

---

# 2) Contraintes de taille et raisons historiques

* **Taille minimale** (de **MAC dest** → **FCS**) :

  * **Untagged** : **64 octets** = 14 (en-tête) + 46 (payload mini) + 4 (FCS)
  * **Tagged 802.1Q** : **68 octets** (4 de plus pour le tag)
* **Pourquoi un minimum ?** Héritage CSMA/CD (Ethernet half-duplex) :

  * Assurer que la trame est **encore en émission** quand une **collision** potentielle revient en écho (détection fiable).
  * Même si aujourd’hui on est quasi tout le temps en **full-duplex**, la **règle de taille mini** reste.
* **Taille maximale standard** :

  * **1518** (sans VLAN) / **1522** (avec 802.1Q).
  * **Jumbo frames** (non standard) : souvent **\~9000 octets** (doivent être **acceptées de bout en bout**).

---

# 3) Exemple concret (annoté)

**ARP broadcast** (qui “demande” l’adresse MAC d’une IP locale) :

```
MAC Dest   : ff ff ff ff ff ff    (broadcast)
MAC Src    : 00 11 22 33 44 55
EtherType  : 08 06                 (ARP)
Payload    : 28 octets (ARP) + 18 octets de padding (zéros) = 46 octets
FCS        : xx xx xx xx           (CRC-32 calculé par la NIC)
```

* Ici l’ARP “pur” ne fait que **28 octets** → on **padde** à **46**.
* Le **switch** diffuse la trame sur tous les ports du VLAN (broadcast).

---

# 4) Variante 802.3 + LLC/SNAP (pour mémoire)

Si le champ après MAC Src est une **longueur** (≤ 1500) :

* On trouve ensuite **LLC** (DSAP, SSAP, Control) puis **SNAP** (OUI + PID) pour identifier le protocole.
* Moins courant que **Ethernet II** aujourd’hui.

---

# 5) Réception : du fil jusqu’à l’IP (vue “pipeline”)

1. **L1** : la NIC voit **Préambule (0x55×7)** → se **synchronise**.
2. **L1** : **SFD (0xD5)** → début de trame.
3. **L2/NIC** :

   * Lit **MAC dest** → si unicast ≠ moi et pas promiscuité → **ignore**.
   * Si pour moi/multicast/broadcast → continue.
   * **Décode VLAN** (si `0x8100/0x88A8`) → PCP/DEI/VID.
   * Lit **EtherType** → sait quel protocole supérieur (IP/ARP/IPv6…).
   * Vérifie **FCS (CRC-32)** → si faux, **jette** la trame.
4. **Pilote** : remet **le payload L3** (ex. IP) à la **couche réseau** de l’OS.

---

## Table rapide des EtherTypes courants

* `0x0800` : **IPv4**
* `0x86DD` : **IPv6**
* `0x0806` : **ARP**
* `0x8100` : **802.1Q** (VLAN tag TPID)
* `0x88A8` : **802.1ad** (QinQ S-Tag)
* `0x8847/0x8848` : **MPLS**
* `0x8863/0x8864` : **PPPoE** (Discovery/Data)
* `0x0805` : X.25, etc. (héritage)

---

## À retenir (ultra-synthèse)

* **Préambule + SFD** : synchro physique (L1).
* **MAC dest / MAC src** : adressage L2.
* **(Option) 802.1Q** : priorité + VLAN ID.
* **EtherType** : indique le protocole L3/L4/L5 transporté.
* **Payload** : 46–1500 octets (padding si < 46).
* **FCS (CRC-32)** : intégrité.
* **IFG** : 96 bit-times entre trames (hors trame).

Si tu veux, je peux te préparer un **schéma annoté** (une vue “boîte” alignée sur un hexdump) et un **exemple calculé** de CRC étape par étape (pédago, sans t’obliger à le recoder).
