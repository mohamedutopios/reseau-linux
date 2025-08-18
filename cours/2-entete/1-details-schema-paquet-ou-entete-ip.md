Top ! Le schéma montre la **structure d’un paquet IPv4** (couche Réseau). On lit l’en-tête de **gauche à droite, puis ligne par ligne**. Voici **chaque champ**, sa **taille** et surtout **à quoi il sert** — puis ce qui se passe en vrai sur le réseau (fragmentation, TTL, etc.).

---

# Les champs de l’en-tête IPv4 (20 à 60 octets)

1. **version (4 bits)**
   → Toujours **4** pour IPv4 (IPv6 = 6). Permet au récepteur d’interpréter le format.

2. **longueur entête / IHL (Internet Header Length) (4 bits)**
   → Taille de l’en-tête en **mots de 32 bits** (multiples de 4 octets).

* **Min = 5** → **20 octets** (sans options).
* **Max = 15** → **60 octets** (avec options).
* Taille en octets = IHL × 4.

3. **type de service (8 bits)**
   → Historiquement ToS ; aujourd’hui utilisé comme **DSCP (6 bits)** pour la QoS + **ECN (2 bits)** pour la gestion de congestion.

* Ex. DSCP=46 (EF pour VoIP), ECN=10 (indique congestion marquée).

4. **longueur totale (16 bits, en octets)**
   → Taille **en-tête + données** (jusqu’à **65 535**).

* Exemple courant sur Ethernet : **≤ 1500** (MTU).
* Sert à savoir où se termine le paquet IP.

5. **identification (16 bits)**
   → Numéro commun à **tous les fragments** d’un même paquet, pour la **réassemblage** à l’arrivée.

* Si DF=1 (voir Flags), certains OS mettent 0.

6. **drapeau (3 bits)** = **Flags**

* bit 0 : **réservé** (=0)
* bit 1 : **DF** (*Don’t Fragment*) — **1** = interdiction de fragmenter.
* bit 2 : **MF** (*More Fragments*) — **1** = il y a **d’autres fragments** après celui-ci (0 pour le dernier/unique).

7. **place du fragment (13 bits)** = **Fragment Offset**
   → Position du fragment **en unités de 8 octets** dans le paquet d’origine.

* Le premier fragment a offset **0** ; les suivants : 8, 16, 24…
* Permet d’assembler dans le bon ordre.

8. **durée de vie (TTL, 8 bits)**
   → Compteur de **sauts**. Chaque routeur **décrémente de 1** ; à **0**, le paquet est détruit et un **ICMP Time Exceeded** est renvoyé.

* Valeurs initiales typiques : 64, 128, 255.

9. **protocole (8 bits)**
   → Indique le **contenu** transporté (la couche au-dessus d’IP) :

* **6** = TCP, **17** = UDP, **1** = ICMP, **47** = GRE, **50** = ESP (IPsec), **89** = OSPF, etc.
* C’est l’équivalent “EtherType” à l’intérieur d’IP.

10. **checksum (16 bits)** = **Header Checksum**
    → Somme de contrôle **de l’en-tête IP uniquement** (pas des données).

* Recalculé à **chaque routeur** (le TTL change → checksum change).
* Si invalide : paquet jeté.

11. **adresse de la source (32 bits)**
    → IPv4 source (ex. 192.168.10.1). Peut être modifiée par **NAT** en chemin.

12. **adresse de la destination (32 bits)**
    → IPv4 destination.

13. **(options)** (0 à 40 octets)
    → Rare en pratique (route enregistrée, timestamp, source routing, sécurité…).

* Les options ont un format **Type/Longueur/Données**.
* **PAD (bourrage)** à **0x00** ajouté si besoin pour que l’IHL soit multiple de 4.

14. **données (payload)**
    → Le **segment TCP**, le **datagramme UDP**, un **message ICMP**, etc.

* La taille utile = **longueur totale − (IHL×4)**.
* Doit respecter la **MTU** du lien (sinon fragmentation, si DF=0).

---

# Ce qui se passe “en vrai” sur le réseau

* **Encapsulation** : Paquet IP (L3) est mis **dans une trame L2** (Ethernet…) et **au-dessus** on trouve TCP/UDP (L4) + l’application (L7).
* **TTL & traceroute** : chaque routeur **décrémente** TTL ; `traceroute`/`tracert` l’exploite pour lister les sauts.
* **Fragmentation** : si un paquet dépasse la **MTU** d’un lien :

  * Si **DF=0** : un routeur **fragmente** → plusieurs paquets avec même **Identification**, **MF=1** sauf le dernier, **Offsets** non nuls.
  * Si **DF=1** : **pas de fragmentation** → le routeur jette et renvoie **ICMP “Fragmentation needed”** (PMTUD).
  * La **réassemblage** n’a lieu **qu’à la destination**.
* **Checksum** : ne couvre **pas** le payload (la fiabilité de bout en bout est assurée par TCP/UDP si besoin).

---

# Petit exemple chiffré (cas simple, sans options)

* IHL = **5** → en-tête = **20 octets**
* Longueur totale = **60** → **données = 40** octets
* TTL = **64**, Protocole = **6 (TCP)**, Flags DF=1, MF=0, Offset=0
  → Paquet non fragmentable (DF), qui transporte un segment TCP de 40 octets. Chaque routeur : TTL 63, 62… et **recalcule** le checksum à chaque saut.

---

# À retenir

* **IHL** donne la taille de l’en-tête (20–60 octets).
* **Total Length** = en-tête + données (max 65 535).
* **TTL** évite les boucles ; **Protocol** indique TCP/UDP/ICMP…
* **Checksum** = en-tête seulement.
* **Flags/Offset** gèrent la **fragmentation**.
* **Options + PAD** existent mais sont rares (et peuvent faire ralentir/filtrer certains équipements).

Si tu veux, je peux te **survoler un paquet réel** (hexdump ou capture Wireshark) et **pointer champ par champ** pour que tu voies comment on lit ces valeurs dans la vraie vie.
