Parfait — je reprends **ton exemple 110.17.14.8/23** et j’ajoute une **méthode infaillible** + **beaucoup d’exemples variés** (avec calculs).
Objectif : à la fin tu sais calculer **réseau, broadcast, plage utilisable, nombre d’hôtes** pour n’importe quel CIDR.

---

# 🧠 Rappel ultra-clair

* **/N** = N bits à 1 dans le masque (bits **réseau**).
* Les bits restants (32–N) = bits **hôtes**.
* **Adresse réseau** : bits hôtes à 0.
* **Broadcast** : bits hôtes à 1.
* **Plage utilisable** : de (réseau +1) à (broadcast –1).
* **Nb d’hôtes** : `2^(32−N) − 2` (sauf /31 et /32, cas particuliers).

---

# 🔧 Deux méthodes de calcul

## 1) Méthode « binaire »

On écrit IP et masque en binaire et on fait un **ET logique** (AND) pour obtenir l’adresse réseau. Puis, on met tous les bits hôtes à 1 pour le broadcast.

## 2) Méthode « taille de bloc » (rapide, idéale en exam)

1. Trouve l’**octet intéressant** (celui où s’arrête le /N).
2. Calcule la **taille de bloc** : `256 − (octet du masque)`.
3. Dans l’IP, prends l’**octet intéressant** et repère **la tranche** :

   * `plancher = ⌊octetIP / bloc⌋ × bloc` ⇒ **octet réseau**
   * `plafond  = plancher + bloc − 1` ⇒ **octet broadcast**
4. Tous les octets **à gauche** restent identiques à l’IP,
   ceux **à droite** sont **0** pour le réseau, **255** pour le broadcast.

---

# ✅ Ton exemple détaillé : **110.17.14.8/23**

* `/23` = masque **255.255.254.0**
  (23 = 16 + 7 → l’octet “intéressant” est le 3ᵉ, **254**)

### Méthode binaire (comme sur ta capture)

IP
`110.17.14.8  = 01101110.00010001.00001110.00001000`
Masque /23
`=            11111111.11111111.11111110.00000000`
**Réseau (AND)**
`=            01101110.00010001.00001110.00000000` → **110.17.14.0**
**Broadcast** : bits hôtes à 1
`=            01101110.00010001.00001111.11111111` → **110.17.15.255**

* **Plage utilisable** : 110.17.14.1 → 110.17.15.254
* **Hôtes** : 2⁹ − 2 = **510** (9 bits hôtes)
* **Taille de bloc (3ᵉ octet)** : 256 − 254 = **2** → les réseaux /23 sautent de **2** en 2 :
  … 110.17.12.0/23, **110.17.14.0/23**, 110.17.16.0/23, …

---

# 🧪 Exemples variés (avec calculs)

## 1) **192.168.1.130/25**

* /25 = **255.255.255.128** → octet intéressant = **4ᵉ**, bloc = **128** (256−128)
* Tranches : 0–127, **128–255** → 130 ∈ \[128–255]
* **Réseau** : 192.168.1.**128**
* **Broadcast** : 192.168.1.**255**
* **Plage** : 192.168.1.129 → 192.168.1.254
* **Hôtes** : 2⁷ − 2 = **126**

> Astuce : /25 = “moitié d’un /24”.

---

## 2) **172.16.100.77/26**

* /26 = **255.255.255.192** → octet intéressant = 4ᵉ, bloc = **64**
* Tranches : 0–63, **64–127**, 128–191, 192–255 → 77 ∈ \[64–127]
* **Réseau** : 172.16.100.**64**
* **Broadcast** : 172.16.100.**127**
* **Plage** : 172.16.100.65 → 172.16.100.126
* **Hôtes** : 2⁶ − 2 = **62**

---

## 3) **172.16.100.77/27** (même IP, sous-réseau plus petit)

* /27 = **255.255.255.224** → bloc = **32**
* Tranches : 0–31, 32–63, **64–95**, 96–127, …
* **Réseau** : 172.16.100.**64**
* **Broadcast** : 172.16.100.**95**
* **Plage** : 172.16.100.65 → 172.16.100.94
* **Hôtes** : 2⁵ − 2 = **30**

> Comparaison /26 vs /27 : on **divise par 2** le nombre d’hôtes (62 → 30) et on **double** le nombre de sous-réseaux.

---

## 4) **10.0.5.10/24**

* /24 = **255.255.255.0** → octet intéressant = 4ᵉ, bloc = **256**
* **Réseau** : 10.0.5.**0**
* **Broadcast** : 10.0.5.**255**
* **Plage** : 10.0.5.1 → 10.0.5.254
* **Hôtes** : 2⁸ − 2 = **254**

> /24 = “un classique” (un octet entier pour les hôtes).

---

## 5) **10.20.30.40/20** (passage dans l’octet 3)

* /20 = **255.255.240.0** → octet intéressant = **3ᵉ**, bloc = **16**
* Tranches (3ᵉ octet) : 0–15, 16–31, **32–47**, … **Mais** notre 3ᵉ octet = **30**, donc tranche **16–31**.
* **Réseau** : 10.20.**16**.0
* **Broadcast** : 10.20.**31**.255
* **Plage** : 10.20.16.1 → 10.20.31.254
* **Hôtes** : 2¹² − 2 = **4094**

> Règle : quand le /N coupe l’**octet 3**, on regarde les **tranches du 3ᵉ octet**.

---

## 6) **203.0.113.19/29**

* /29 = **255.255.255.248** → bloc = **8**
* Tranches : 0–7, 8–15, **16–23**, 24–31, … → 19 ∈ \[16–23]
* **Réseau** : 203.0.113.**16**
* **Broadcast** : 203.0.113.**23**
* **Plage** : 203.0.113.17 → 203.0.113.22
* **Hôtes** : 2³ − 2 = **6**

> Très utilisé pour de **petits segments** (firewall, DMZ, etc.).

---

## 7) **198.51.100.200/30** (lien point-à-point)

* /30 = **255.255.255.252** → bloc = **4**
* Tranches : …, **200–203**, 204–207, …
* **Réseau** : 198.51.100.**200**
* **Broadcast** : 198.51.100.**203**
* **Plage** : 198.51.100.**201–202**
* **Hôtes** : 2² − 2 = **2**

> Parfait pour **P2P** (routeurs).

---

## 8) **100.64.12.250/22**

* /22 = **255.255.252.0** → octet intéressant = 3ᵉ, bloc = **4**
* Tranches 3ᵉ octet : … 8–11, **12–15**, 16–19 …
* **Réseau** : 100.64.**12**.0
* **Broadcast** : 100.64.**15**.255
* **Plage** : 100.64.12.1 → 100.64.15.254
* **Hôtes** : 2¹⁰ − 2 = **1022**

---

## 9) **Cas particuliers /31 et /32**

* **/32** : **255.255.255.255**

  * **Un seul hôte** (ex. loopback 127.0.0.1/32, routes très spécifiques).
  * Pas de réseau/broadcast au sens classique.

* **/31** : **255.255.255.254**

  * 2 adresses (ex. `.0` et `.1`) — **les deux sont utilisables** en **point-à-point** (RFC 3021).
  * Idéal pour économiser des IP sur des liens routeur-routeur.

---

# 🗂️ Mini-aide-mémoire (les plus courants)

| CIDR | Masque          | Hôtes utilisables |
| ---: | --------------- | ----------------- |
|  /30 | 255.255.255.252 | 2                 |
|  /29 | 255.255.255.248 | 6                 |
|  /28 | 255.255.255.240 | 14                |
|  /27 | 255.255.255.224 | 30                |
|  /26 | 255.255.255.192 | 62                |
|  /25 | 255.255.255.128 | 126               |
|  /24 | 255.255.255.0   | 254               |
|  /23 | 255.255.254.0   | 510               |
|  /22 | 255.255.252.0   | 1022              |
|  /21 | 255.255.248.0   | 2046              |
|  /20 | 255.255.240.0   | 4094              |

> Rappel : **Hôtes = 2^(32−CIDR) − 2** (sauf /31 et /32).

---

# 🎓 Astuces pédagogiques & contrôles rapides

* **Taille de bloc** = `256 − masque_octet` (dans l’octet intéressant).
* Pour savoir **dans quelle tranche** tombe un octet : fais `⌊octet/Bloc⌋ × Bloc`.
* **Prochaine tranche** = `octet_réseau + Bloc`.
* **/23** et **/22** : on **regarde le 3ᵉ octet** (très formateur).
* **Sanity check** : en /26, la plage doit finir en `.63/.127/.191/.255` selon la tranche.

---

Si tu veux, je te prépare **une série d’exercices** (10 à 20 IP/CIDR mélangés) avec **corrigés détaillés** au format que tu viens de voir (méthode + calculs + réponse).
