Super question 👌 ! Une **trame Ethernet** est l’unité de transmission de la **couche 2 (liaison de données)** du modèle OSI.
C’est ce que les cartes réseau (NIC) échangent sur un réseau local (LAN).

Voici sa **composition complète** (Ethernet II, la plus utilisée aujourd’hui) :

---

# 🔹 Structure d’une trame Ethernet II

```
+------------+------------+---------+------------------+----------+
| Préambule  | SFD        | En-tête | Données (Payload)| FCS      |
+------------+------------+---------+------------------+----------+
```

---

## 1. Préambule (7 octets)

* Suite de bits `10101010` répétés.
* Sert à **synchroniser l’horloge** entre émetteur et récepteur.
* Non compté dans la longueur officielle de la trame.

---

## 2. SFD – Start Frame Delimiter (1 octet)

* Valeur fixe : `10101011`.
* Indique le **début réel de la trame**.

---

## 3. En-tête Ethernet (14 octets)

* **Adresse MAC destination** (6 octets)

  * Identifie la machine réceptrice (ou broadcast : `FF:FF:FF:FF:FF:FF`).
* **Adresse MAC source** (6 octets)

  * Adresse physique de l’émetteur.
* **Type / Longueur** (2 octets)

  * Dans Ethernet II → **Type de protocole encapsulé** :

    * `0x0800` → IPv4
    * `0x86DD` → IPv6
    * `0x0806` → ARP

*(Dans IEEE 802.3, ce champ indiquait la longueur des données → ≤ 1500 octets.)*

---

## 4. Données (Payload) (46 à 1500 octets)

* Contenu transporté → souvent un **paquet IP**.
* Minimum 46 octets (rembourrage ajouté si nécessaire).
* Maximum 1500 octets (MTU standard).
* Avec **jumbo frames** : jusqu’à 9000 octets.

---

## 5. FCS – Frame Check Sequence (4 octets)

* **CRC-32** (Contrôle d’intégrité).
* Permet de vérifier si la trame est corrompue.
* Si erreur → trame rejetée.

---

# 🔹 Taille totale

* **Minimale** : 64 octets (de Destination MAC jusqu’au FCS, sans compter préambule + SFD).
* **Maximale** : 1518 octets (standard) → 1522 avec VLAN (802.1Q).

---

# 🔹 Exemple visuel avec valeurs

```
Préambule : 7 octets  (10101010…)
SFD       : 1 octet   (10101011)
MAC Dest  : 6 octets  (FF:FF:FF:FF:FF:FF)
MAC Src   : 6 octets  (00:1A:2B:3C:4D:5E)
Type      : 2 octets  (0x0800 → IPv4)
Payload   : 46-1500 octets (ex : paquet IP)
FCS       : 4 octets  (CRC32)
```

---

✅ **Résumé clair** :
Une trame Ethernet contient :

1. **Préambule + SFD** (synchronisation)
2. **MAC destination + MAC source + Type** (en-tête)
3. **Payload (données IP, ARP, etc.)**
4. **FCS (contrôle d’erreur CRC32)**

---

Veux-tu que je te fasse un **schéma en couches** montrant comment un **paquet IP** est encapsulé dans une trame Ethernet complète (L3 → L2) ?
