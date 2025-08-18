Super, voici les **corrections des exercices 1 à 5** avec **plus d’explications** : je détaille la méthode (tri par taille, calcul du masque, “octet d’intérêt”, incrément, bornes réseau/broadcast) et j’explique **pourquoi** chaque valeur.

---

# 🟩 Exercice 1 (192.168.10.0/24)

**Besoins** : A=60, B=30, C=10, D=2 (P2P)
**Principe VLSM** : on trie par taille ↓, on calcule le plus petit masque possible, on **place en premier le plus gros** pour éviter les chevauchements.

## 1) Choix des masques (hôtes = $2^n - 2$)

* A (60) → $2^6-2=62$ ⇒ **/26** (255.255.255.192) → bloc de 64 adresses (62 utilisables)
* B (30) → $2^5-2=30$ ⇒ **/27** (255.255.255.224) → bloc de 32 adresses
* C (10) → $2^4-2=14$ ⇒ **/28** (255.255.255.240) → bloc de 16 adresses
* D (2)  → $2^2-2=2$  ⇒ **/30** (255.255.255.252) → bloc de 4 adresses (2 utilisables)

> **Astuce “octet d’intérêt & incrément”** (ici, tout se joue au **4ᵉ octet** car /26, /27, /28, /30 > /24) :
> Incrément = $256 - \text{valeur du masque dans l’octet}$
> /26 → 256−192=**64** ; /27 → **32** ; /28 → **16** ; /30 → **4**.
> Les réseaux commencent **sur des multiples de l’incrément**.

## 2) Placement (en partant de 192.168.10.0)

### A — /26 (incrément 64)

* Réseau : **192.168.10.0/26** (multiple de 64)
* Hôtes : 192.168.10.1 → 192.168.10.62
* Broadcast : 192.168.10.63

### B — /27 (incrément 32)

* Prochain bloc aligné après .63 = **.64**
* Réseau : **192.168.10.64/27**
* Hôtes : 192.168.10.65 → 192.168.10.94
* Broadcast : 192.168.10.95

### C — /28 (incrément 16)

* Prochain bloc après .95 = **.96**
* Réseau : **192.168.10.96/28**
* Hôtes : 192.168.10.97 → 192.168.10.110
* Broadcast : 192.168.10.111

### D — /30 (incrément 4)

* Prochain bloc après .111 = **.112**
* Réseau : **192.168.10.112/30**
* Hôtes : 192.168.10.113 → 192.168.10.114
* Broadcast : 192.168.10.115

**Reste libre** : 192.168.10.116 → 192.168.10.255

> **Pourquoi ces bornes ?**
> Broadcast = **(réseau du bloc suivant) − 1**.
> Comme les blocs commencent à .0, .64, .96, .112, etc. on soustrait 1.

---

# 🟨 Exercice 2 (172.16.0.0/16)

**Besoins** : A=1000, B=500, C=200, D=50
Ici, l’**octet d’intérêt** peut être le **3ᵉ** (car /22 et /23 touchent le 3ᵉ octet).

## 1) Masques

* A (1000) → $2^{10}-2=1022$ ⇒ **/22** (255.255.252.0) → **incrément 4** au 3ᵉ octet
* B (500) → $2^9-2=510$   ⇒ **/23** (255.255.254.0) → **incrément 2** au 3ᵉ octet
* C (200) → $2^8-2=254$   ⇒ **/24** (255.255.255.0) → **incrément 1** au 3ᵉ octet
* D (50)  → $2^6-2=62$    ⇒ **/26** (255.255.255.192) → **incrément 64** au 4ᵉ octet

> **Pourquoi “incrément au 3ᵉ octet” pour /22 ?**
> /22 = 255.255.**252**.0 → 256 − 252 = **4** → les sous-réseaux commencent à 172.16.**0**.0, 172.16.**4**.0, 172.16.**8**.0, …

## 2) Placement

### A — /22

* Réseau : **172.16.0.0/22** (3ᵉ octet multiple de 4)
* Hôtes : 172.16.0.1 → 172.16.3.254
* Broadcast : 172.16.3.255

### B — /23

* Prochain multiple de 2 au 3ᵉ octet après 3 = **4**
* Réseau : **172.16.4.0/23**
* Hôtes : 172.16.4.1 → 172.16.5.254
* Broadcast : 172.16.5.255

### C — /24

* Prochain /24 après .5 = **.6**
* Réseau : **172.16.6.0/24**
* Hôtes : 172.16.6.1 → 172.16.6.254
* Broadcast : 172.16.6.255

### D — /26 (octet d’intérêt = 4ᵉ, incrément 64)

* Prochain bloc après .6.255 = **172.16.7.0**
* Réseau : **172.16.7.0/26**
* Hôtes : 172.16.7.1 → 172.16.7.62
* Broadcast : 172.16.7.63

**Reste** : 172.16.7.64 → 172.16.255.255

> **Vérif alignement** :
> Pour un /23, le 3ᵉ octet doit être pair (multiples de 2) : 0, 2, 4, 6, …
> Pour un /22, multiples de 4 : 0, 4, 8, …

---

# 🟥 Exercice 3 (10.0.0.0/24)

**Besoins** : A=100, B=50, C=25, D/E/F = 3×P2P

## 1) Masques et incréments (4ᵉ octet)

* A → /25 (incrément **128**)
* B → /26 (incrément **64**)
* C → /27 (incrément **32**)
* D/E/F → /30 (incrément **4**)

## 2) Placement

### A — /25

* **10.0.0.0/25**
* Hôtes : 10.0.0.1 → 10.0.0.126
* Broadcast : 10.0.0.127

### B — /26

* Prochain multiple de 64 après 127 = **128**
* **10.0.0.128/26**
* Hôtes : 10.0.0.129 → 10.0.0.190
* Broadcast : 10.0.0.191

### C — /27

* Prochain multiple de 32 après 191 = **192**
* **10.0.0.192/27**
* Hôtes : 10.0.0.193 → 10.0.0.222
* Broadcast : 10.0.0.223

### D/E/F — /30

* /30 #1 : **10.0.0.224/30** → hôtes .225–.226, broadcast .227
* /30 #2 : **10.0.0.228/30** → hôtes .229–.230, broadcast .231
* /30 #3 : **10.0.0.232/30** → hôtes .233–.234, broadcast .235

**Reste** : 10.0.0.236 → 10.0.0.255

> **Pourquoi /30 pour P2P ?**
> Un /30 contient 4 adresses : réseau, 2 hôtes, broadcast. Parfait pour **2 extrémités de routeurs**.
> (En prod, **/31** existe pour P2P et n’a pas de broadcast, cf. RFC 3021.)

---

# 🟦 Exercice 4 (192.168.20.0/27)

**Question** : hôtes, plage, broadcast, prochain réseau.

* /27 ⇒ $n = 32 - 27 = 5$ bits hôtes → $2^5 - 2 = 30$ hôtes utilisables
* Masque = 255.255.255.224 → **incrément 32** au 4ᵉ octet
* Réseau : **192.168.20.0/27** (car .0 est multiple de 32)
* Hôtes : **192.168.20.1 → 192.168.20.30**
* Broadcast : **192.168.20.31**
* **Prochain réseau** (/27) : **192.168.20.32/27** (puis .64/27, .96/27, …)

> **Règle** : pour trouver le **broadcast**, prends le **début du bloc suivant** (prochain multiple de l’incrément) et **soustrais 1**.

---

# 🟪 Exercice 5 (192.168.1.0/24)

**Besoins** :

* 1×100 hôtes → **/25** (128 adresses)
* 2×50 hôtes → **2×/26** (2×64 = 128 adresses)
* 3×10 hôtes → **3×/28** (3×16 = 48 adresses)

## 1) Test de faisabilité rapide

Somme des **blocs** (pas des hôtes) dans un /24 :

* Demande = 128 + 128 + 48 = **304 adresses** > **256** → ❌ **Impossible** dans **un** /24.

> **Autre manière de voir** (en “fractions” de /24) :
>
> * 1×/25 = **1/2** du /24
> * 2×/26 = **2×1/4 = 1/2**
> * 3×/28 = **3×1/16 = 3/16**
>   Total = **1/2 + 1/2 + 3/16 = 1 + 3/16 = 1,1875** (> 1) ⇒ dépasse un /24.

## 2) Deux options

* **Élargir** l’espace à **/23** (512 adresses), par ex. 192.168.0.0/23
* **Réduire** les besoins

### Proposition correcte avec **192.168.0.0/23**

* /23 couvre **192.168.0.0 → 192.168.1.255**

On place les blocs **alignés** :

| Réseau | Adresse réseau   | Masque          | Plage hôtes                   | Broadcast     | Pourquoi ici ?                        |
| -----: | ---------------- | --------------- | ----------------------------- | ------------- | ------------------------------------- |
|    100 | 192.168.0.0/25   | 255.255.255.128 | 192.168.0.1 – 192.168.0.126   | 192.168.0.127 | /25 → incrément 128, démarre à .0     |
|  50 #1 | 192.168.0.128/26 | 255.255.255.192 | 192.168.0.129 – 192.168.0.190 | 192.168.0.191 | /26 → incrément 64, après .127 ⇒ .128 |
|  50 #2 | 192.168.0.192/26 | 255.255.255.192 | 192.168.0.193 – 192.168.0.254 | 192.168.0.255 | Prochain multiple de 64 ⇒ .192        |
|  10 #1 | 192.168.1.0/28   | 255.255.255.240 | 192.168.1.1 – 192.168.1.14    | 192.168.1.15  | /28 → incrément 16, nouveau /24 dispo |
|  10 #2 | 192.168.1.16/28  | 255.255.255.240 | 192.168.1.17 – 192.168.1.30   | 192.168.1.31  | +16                                   |
|  10 #3 | 192.168.1.32/28  | 255.255.255.240 | 192.168.1.33 – 192.168.1.46   | 192.168.1.47  | +16                                   |

**Reste libre** : 192.168.1.48 → 192.168.1.255 (beaucoup d’espace)

---

## 🧠 Mémo méthode (utile partout)

1. **Trier** les besoins (du plus grand au plus petit).
2. Pour chaque besoin, trouver le **plus petit n** tel que $2^n-2 \ge \text{hôtes}$, puis **CIDR = / (32−n)**.
3. **Identifier l’octet d’intérêt** (où s’arrête le préfixe) et calculer **l’incrément** : $256 - \text{masque de cet octet}$.
4. **Aligner** le début du bloc sur un **multiple de l’incrément**.
5. **Plage hôtes** = $[réseau+1, \text{broadcast}-1]$.
6. **Broadcast** = $(\text{début du bloc suivant}) - 1$.

> ⚠️ **Pièges classiques** :
>
> * Oublier que **réseau** et **broadcast** ne sont pas attribuables.
> * Ne pas **aligner** sur l’incrément → chevauchements.
> * Mélanger l’**octet d’intérêt** (3ᵉ pour /21–/23, 4ᵉ pour /25–/30, etc.).

Si tu veux, je te fais un \*\* mémo imprimable\*\* (tableau des masques ↔ incréments ↔ hôtes) ou un **schéma Packet Tracer** basé sur un des exos.
