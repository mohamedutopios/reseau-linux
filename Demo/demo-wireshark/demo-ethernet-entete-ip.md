Excellente question 👍 Tu vas voir que **Wireshark** est l’outil parfait pour “disséquer” une trame Ethernet et un en-tête IP.
Je vais t’expliquer **pas à pas**, avec un exemple concret.

---

## ⚙️ Étape 1 : Capturer du trafic

1. Lance **Wireshark**.
2. Choisis l’interface réseau active (ex. ta carte Ethernet ou Wi-Fi).
3. Clique sur **Start Capturing Packets** (l’icône requin bleu).
4. Génère du trafic simple (ouvre un site web, ping une IP…).

---

## ⚙️ Étape 2 : Filtrer ce que tu veux analyser

* Pour ne voir **que les trames Ethernet** :

  ```
  eth
  ```
* Pour ne voir **que les paquets IP** :

  ```
  ip
  ```
* Pour un **ping ICMP** (facile à reconnaître) :

  ```
  icmp
  ```
* Pour **un port TCP spécifique** (ex. HTTP sur 80) :

  ```
  tcp.port == 80
  ```

---

## ⚙️ Étape 3 : Lire la trame dans Wireshark

Quand tu sélectionnes une trame dans la liste du haut, Wireshark l’analyse en **3 zones** :

1. **Frame** (en haut de la partie du milieu)
   → Infos générales (n° de trame, taille capturée, timestamp).

2. **Ethernet II**

   * **Destination MAC** (ex. `68:1d:ef:40:90:9b`)
   * **Source MAC** (ex. `00:1a:2b:3c:4d:5e`)
   * **Type** (0x0800 = IPv4, 0x86dd = IPv6, 0x0806 = ARP)

3. **Internet Protocol (IP)**

   * Version (IPv4 ou IPv6)
   * Header length (IHL)
   * Differentiated Services (DSCP/ECN)
   * Total Length
   * Identification, Flags, Fragment offset
   * TTL
   * Protocol (6 = TCP, 17 = UDP, 1 = ICMP)
   * Header checksum
   * Source IP / Destination IP

4. **Transport (TCP/UDP/ICMP)**

   * Ports source/destination
   * Flags TCP (SYN, ACK, FIN, etc.)
   * Checksum UDP/TCP

5. **Data (Payload)**
   → Contenu brut de l’application (HTTP, DNS, etc.)

---

## ⚙️ Étape 4 : Exemple concret (Ping ICMP)

Si tu fais un `ping 8.8.8.8`, tu verras :

* **Ethernet II**

  * Src MAC : ta carte réseau
  * Dst MAC : passerelle (si en dehors de ton LAN)

* **Internet Protocol v4**

  * Src IP : ton PC (ex. `192.168.1.10`)
  * Dst IP : `8.8.8.8`
  * TTL : 128 (Windows) ou 64 (Linux/Mac)
  * Protocol : ICMP (1)

* **Internet Control Message Protocol**

  * Type 8 (Echo Request) → aller
  * Type 0 (Echo Reply) → retour

---

## ⚙️ Étape 5 : Voir la trame brute (hexa)

En bas, Wireshark montre le **dump hexadécimal** de la trame.
Exemple (simplifié) :

```
ff ff ff ff ff ff 00 1a 2b 3c 4d 5e 08 00 45 00
00 3c 1c 46 40 00 40 01 72 b7 c0 a8 01 0a 08 08
08 08 08 00 4d 5c 00 01 00 02 61 62 63 64 ...
```

* `ff ff ff ff ff ff` = MAC destination (broadcast)
* `00 1a 2b 3c 4d 5e` = MAC source
* `08 00` = EtherType (IPv4)
* `45` = Version 4 + IHL=5
* etc.

👉 Tu peux cliquer sur chaque champ dans le milieu, et Wireshark **surligne la portion correspondante** dans le dump hexadécimal.

---

✅ Avec ça, tu peux **analyser complètement une trame Ethernet et un en-tête IP**.
C’est exactement ce qu’on fait en **cours réseaux ou pentest** pour vérifier la structure et le contenu des paquets.

---

Veux-tu que je te prépare un **exemple annoté** (paquet complet : Ethernet + IPv4 + TCP avec hexa et interprétation champ par champ), comme si tu avais capturé un petit `curl` ou `ping` dans Wireshark ?
