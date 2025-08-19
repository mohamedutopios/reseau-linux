Parfait — je reprends **exactement tes 7 exemples `traceroute`** et j’augmente le niveau d’explication : **mécanique interne (TTL, types ICMP), décodage ligne par ligne, et variantes utiles** pour ton lab **3 VMs**.

> Contexte lab :
> **vm1 = 10.10.10.11**, **vm2 = 10.10.10.12**, **vm3 = 10.10.10.13** (toutes sur `enp0s8`, réseau 10.10.10.0/24)

---

## Rappel express : comment `traceroute` fonctionne (UDP par défaut)

* En **mode UDP** (par défaut sous Linux), `traceroute` envoie 3 paquets UDP (par “saut”) vers la cible, avec **TTL** (Time To Live) croissant : 1, 2, 3, …
* Chaque routeur qui voit passer un paquet dont le TTL arrive à 0 renvoie un **ICMP Time Exceeded** (type 11, code 0). C’est comme ça qu’on voit chaque “saut”.
* Quand le paquet **atteint la cible**, rien n’écoute sur les ports UDP élevés (33434, 33435, 33436…), donc la cible renvoie **ICMP Destination Unreachable – Port Unreachable** (type 3, code 3). C’est le **dernier saut**.
* En **mode ICMP** (`-I`) : on envoie des ICMP Echo Request (comme `ping`), on attend des Echo Reply.
* En **mode TCP** (`-T`) : on envoie des SYN vers un port (`-p`), on mesure les réponses (SYN/ACK, RST) ou l’absence de réponse.

---

# 🟦 1) Traceroute simple dans le LAN (vm1 → vm2)

**Commande (vm1)**

```bash
traceroute 10.10.10.12
```

**Exemple de sortie**

```
traceroute to 10.10.10.12 (10.10.10.12), 30 hops max, 60 byte packets
 1  10.10.10.12  0.345 ms  0.280 ms  0.310 ms
```

**Décodage détaillé**

* `traceroute to …, 30 hops max` : on ira au plus jusqu’à TTL=30.
* `60 byte packets` : taille du paquet envoyé (en L3/L4, indicatif).
* `1` : **premier saut**.
* `10.10.10.12` : c’est **vm2**. En LAN, il n’y a **aucun routeur** → on arrive direct.
* Les **trois temps** sont 3 mesures RTT (aller-retour) pour 3 paquets.

**Ce qui se passe vraiment**

* `traceroute` envoie **3 UDP** (ports 33434-33436) avec **TTL=1**.
* Aucun routeur à franchir : la cible reçoit les UDP et répond par **ICMP port unreachable (3/3)**.
* `traceroute` l’affiche comme **saut 1** (dernier saut).

**À montrer aux élèves**

* **LAN = 1 saut**. Latences sous le milliseconde = normal en virtuel.
* Ajoute `-n` pour éviter les résolutions DNS et accélérer l’affichage :

  ```bash
  traceroute -n 10.10.10.12
  ```

---

# 🟦 2) Traceroute (vm1 → vm3, même LAN)

**Commande (vm1)**

```bash
traceroute 10.10.10.13
```

**Exemple de sortie**

```
traceroute to 10.10.10.13 (10.10.10.13), 30 hops max, 60 byte packets
 1  10.10.10.13  0.350 ms  0.320 ms  0.340 ms
```

**Décodage**

* Identique au cas précédent : **1 saut**, c’est la cible.
* Les trois RTT sont très proches (jitter quasi nul en lab virtuel).

**Variante utile**

* Montre la différence **UDP vs ICMP** :

  ```bash
  traceroute -I 10.10.10.13   # ICMP Echo à la place de UDP
  ```

  Si un pare-feu bloque ICMP Echo Reply, ce mode échouera (voir §4).

---

# 🟦 3) Traceroute WAN (vm1 → 8.8.8.8)

**Prérequis**
Une route par défaut (ex. `default via 10.10.10.1`) et de la sortie Internet.

**Commande (vm1)**

```bash
traceroute 8.8.8.8
```

**Exemple de sortie**

```
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  10.10.10.1     1.150 ms  1.120 ms  1.130 ms
 2  192.168.1.254  3.200 ms  3.250 ms  3.180 ms
 3  8.8.8.8       20.300 ms 20.350 ms 20.270 ms
```

**Décodage**

* **Saut 1** : ta passerelle locale (box/VM NAT). Temps \~1 ms.
* **Saut 2** : routeur intermédiaire (FAI / réseau amont).
* **Saut 3** : la cible (8.8.8.8). Temps plus élevé (WAN).

**À expliquer**

* **TTL** : au saut n, on envoie des paquets TTL=n.
* Chaque routeur décrémente TTL et renvoie **ICMP Time Exceeded** quand TTL=0.
* La **latence augmente** naturellement à chaque saut.

**Variante utile**

* Sans reverse-DNS (plus rapide, évite d’attendre les noms) :

  ```bash
  traceroute -n 8.8.8.8
  ```

---

# 🟦 4) Traceroute avec pertes / filtrage (ICMP bloqué sur vm2)

## 4.A — Bloquer **ICMP Echo** (impacte `traceroute -I` et `ping`)

**Règle (vm2)**

```bash
sudo iptables -I INPUT -p icmp -j DROP
```

* `-I INPUT` : insère en tête de chaîne **INPUT** (trafic entrant).
* `-p icmp` : cible le protocole ICMP.
* `-j DROP` : on **jette** silencieusement.

**Effets**

* `ping vm2` échoue.
* `traceroute -I 10.10.10.12` (mode ICMP) échoue.

**Test (vm1)**

```bash
traceroute -I 10.10.10.12
```

**Exemple de sortie**

```
 1  * * *
```

* `*` = **pas de réponse** (timeout). On a envoyé, rien n’est revenu.

**À comprendre**

* Ici tu bloques les **Echo Reply** (réponse à ICMP Echo).
* **Mais** le `traceroute` par défaut (UDP) peut **encore** réussir, cf. 4.B.

## 4.B — Casser le `traceroute` **UDP par défaut**

`traceroute` (UDP) a besoin que la **cible** réponde avec **ICMP Port Unreachable (3/3)**.
Tu peux casser ça de deux manières (sur **vm2** la cible) :

### Option 1 — Bloquer **les réponses ICMP port-unreachable** en **sortie**

```bash
sudo iptables -I OUTPUT -p icmp --icmp-type port-unreachable -j DROP
```

* Cette règle empêche vm2 d’envoyer le message “port unreachable”.
* Résultat : `traceroute` UDP **ne voit plus le dernier saut**.

### Option 2 — Bloquer **les paquets UDP entrants** sur les **ports 33434+**

```bash
sudo iptables -I INPUT -p udp --dport 33434:33534 -j DROP
```

* Tu empêches la cible de **recevoir** les UDP du traceroute.
* Résultat : pas de réponse finale non plus.

**Vérifie l’effet côté vm1**

```bash
traceroute 10.10.10.12
# => selon la règle appliquée, tu verras soit des * soit un inachevé
```

**RECOMMANDATION PÉDAGO**
Montre aux élèves que **bloquer “ICMP Echo”** ne suffit pas à casser **tous** les traceroute :

* `traceroute -I` (ICMP) : KO
* `traceroute` (UDP) : parfois encore OK
* `traceroute -T -p 22` (TCP) : peut être OK si 22 n’est pas filtré

## 4.C — DROP vs REJECT (différence visible)

**Remplace la règle** pour **REJECT** (vm2)

```bash
sudo iptables -R INPUT 1 -p icmp -j REJECT
```

* `-R INPUT 1` : remplace la 1ʳᵉ règle de la chaîne `INPUT`.
* `REJECT` : renvoie un message ICMP d’erreur (ex. administratively prohibited).

**Effet côté vm1 (ping)**

* Avec **DROP** : timeouts.
* Avec **REJECT** : message explicite (“Destination… Unreachable”).
  Pédago : **“silence” vs “non”** — très utile au diagnostic.

**Remettre propre**

```bash
sudo iptables -D INPUT -p icmp -j REJECT  # (ou DROP selon ta dernière règle)
sudo iptables -D INPUT -p udp --dport 33434:33534 -j DROP
sudo iptables -D OUTPUT -p icmp --icmp-type port-unreachable -j DROP
```

> Remarque : ces règles sont **volatiles**. Pour persister : `sudo apt install iptables-persistent`.

---

# 🟦 5) Options utiles — avec **explication précise**

* `-n` : **ne résout pas** les noms → plus **rapide**, résultats en **IP** uniquement.
* `-q N` : **N sondes** par saut (par défaut 3).
  *Moins verbeux pour les slides :* `traceroute -q 1 10.10.10.12`
* `-w S` : **timeout** (en secondes) pour chaque sonde.
  *Plus strict/rapide :* `traceroute -w 1 8.8.8.8` (plus d’étoiles si c’est lent)
* `-m H` : **TTL max** (nombre de sauts).
  *Limiter l’affichage :* `traceroute -m 5 8.8.8.8`
* `-I` : mode **ICMP Echo** (utile si UDP bloqué, mais cassable via icmp DROP).
* `-T` : mode **TCP SYN** (utile si ICMP/UDP filtrés).
  *Ex. simuler HTTPS :* `traceroute -T -p 443 google.com`
* `-f X` : **TTL de départ** (par défaut 1).
  *Ex. sauter les premiers routeurs :* `traceroute -f 3 8.8.8.8`
* `-A` : affiche les **AS** (quand supporté) → pédagogique sur le transit.

---

# 🟦 6) Comment lire **et interpréter** les résultats

* **Chaque ligne = un saut** (routeur ou la cible).
* **3 temps** = 3 sondes RTT. Une valeur très différente = **gigue** (jitter) / file d’attente.
* **`*`** = pas de réponse (timeout, filtrage).
* **Latence croissante** : **normal** en s’éloignant.
* **Pics de latence à un saut précis** : routeur saturé, lien congestionné.
* **Codes “!”** (Linux) **sur la dernière colonne** :

  * `!N` : Network unreachable
  * `!H` : Host unreachable
  * `!P` : Protocol unreachable
  * `!F` : Fragmentation needed (DF set)
  * `!X` : Communication administratively prohibited
    Ces codes aident à distinguer **absence de réponse** (étoiles) d’un **refus explicite**.

**Tip diagnostic**

* Si **les premiers sauts répondent** puis `* * *` à partir d’un point → filtrage en milieu de chemin.
* Si **rien ne répond** (même saut 1) → souci local (pare-feu local, pas de route, interface down).

---

# 🟦 7) Mini-scénarios lab (clés en main)

### A) LAN direct (1 saut)

* `traceroute 10.10.10.12` → **1 saut** (cible).
* Variante : `-I` vs UDP pour montrer protocole.

### B) Routage intermédiaire (2+ sauts)

* Fais de **vm2 un routeur** vers `192.168.50.0/24` (NIC2 sur vm2 + `ip_forward=1`).
* `traceroute 192.168.50.10` depuis vm1 → **vm2** puis **hôte** derrière.

### C) Filtrage sélectif (pédago sécurité)

* Sur vm2 : `iptables -I INPUT -p icmp -j DROP` → **KO pour `-I`**,
  mais `traceroute -T -p 22 10.10.10.12` **OK** si 22 ouvert.
* Montre ensuite blocage des **réponses ICMP 3/3** (OUTPUT) pour casser UDP.

### D) Observer “ce qui se passe vraiment” (bonus)

* Sur vm2 :

  ```bash
  sudo tcpdump -ni enp0s8 "icmp or (udp and port 33434)"
  ```
* Relance `traceroute` depuis vm1 et **montre** :

  * Les **UDP** qui arrivent (ports 33434+).
  * Les **ICMP time-exceeded** (routeurs intermédiaires).
  * Les **ICMP port-unreachable** (réponse finale de la cible).

---

## Fiches “prêtes pour slides”

* **Par défaut**, Linux `traceroute` = **UDP** vers ports **33434+** ; **dernier saut = ICMP 3/3**.
* **`-I`** = ICMP Echo ; **`-T -p PORT`** = TCP SYN (utile si ICMP/UDP filtrés).
* **`*`** = timeout/filtrage ; **`!X`/`!H`/…** = refus explicite.
* **`-n`** (pas de DNS) = +rapide, +clair en lab.
* Pour **casser** UDP traceroute : **bloquer l’ICMP 3/3 en sortie** ou les **UDP 33434+ en entrée** sur la **cible**.

---

Si tu veux, je peux te générer un **énoncé TP** + **corrigé** qui enchaîne ces cas (LAN, WAN, filtrage, TCP fallback) avec captures `tcpdump` à l’appui — prêt à imprimer.
