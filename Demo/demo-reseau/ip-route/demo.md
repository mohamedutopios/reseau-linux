Excellent 👍 on complète la trilogie avec **`ip route`**, toujours dans ton **lab 3 VMs** :

* **vm1 = 10.10.10.11**
* **vm2 = 10.10.10.12**
* **vm3 = 10.10.10.13**

⚡ `ip route` sert à afficher, ajouter, supprimer et modifier les routes IP, donc à contrôler **par où passent les paquets**.

Je vais te donner des **démos pédagogiques complètes** (commandes + résultats + explications + contexte).

---

# 🟦 1) Afficher la table de routage

Sur **vm1** :

```bash
ip route
```

📊 Exemple de sortie :

```
default via 10.10.10.1 dev enp0s8
10.10.10.0/24 dev enp0s8 proto kernel scope link src 10.10.10.11
```

🔎 Explication :

* `10.10.10.0/24 dev enp0s8` = réseau local connu directement.
* `default via 10.10.10.1` = tout ce qui n’est pas LAN part vers la passerelle `.1`.

🎯 Dans le lab : si tu n’as pas mis de route par défaut, seul `10.10.10.0/24` apparaît → la VM ne saura pas joindre Internet.

---

# 🟦 2) Ajouter une route statique vers un autre réseau

Imaginons que **vm3** ait une deuxième carte vers un réseau **192.168.50.0/24**.
On veut que **vm1** y accède via vm3.

Sur **vm1** :

```bash
sudo ip route add 192.168.50.0/24 via 10.10.10.13
ip route | grep 192.168.50.0
```

📊 Résultat attendu :

```
192.168.50.0/24 via 10.10.10.13 dev enp0s8
```

🔎 Explication :

* Les paquets destinés à 192.168.50.0/24 seront envoyés à vm3 (qui doit router).

🎯 Démo utile : simuler une VM “routeur” qui connecte 2 réseaux.

---

# 🟦 3) Modifier la route par défaut

Sur **vm1** :

```bash
sudo ip route replace default via 10.10.10.12
ip route | grep default
```

📊 Résultat attendu :

```
default via 10.10.10.12 dev enp0s8
```

🔎 Explication :

* Au lieu d’envoyer le trafic Internet vers `.1`, vm1 utilise maintenant vm2 comme passerelle.

🎯 Scénario réaliste : vm2 joue le rôle de **NAT vers Internet**.

---

# 🟦 4) Supprimer une route

Sur **vm1** :

```bash
sudo ip route del default
ip route
```

📊 Résultat attendu :

```
10.10.10.0/24 dev enp0s8 proto kernel scope link src 10.10.10.11
```

🔎 Explication :

* Sans route `default`, impossible d’atteindre une IP externe (ex: 8.8.8.8).
* Seul le LAN reste accessible.

🎯 Démo claire : montre aux apprenants pourquoi une machine ping son LAN mais pas Internet.

---

# 🟦 5) Routes avec métriques (priorité)

Sur **vm1** :

```bash
sudo ip route add default via 10.10.10.12 metric 200
sudo ip route add default via 10.10.10.13 metric 100
ip route show
```

📊 Résultat attendu :

```
default via 10.10.10.13 dev enp0s8 metric 100
default via 10.10.10.12 dev enp0s8 metric 200
```

🔎 Explication :

* Deux routes par défaut, mais celle avec **metric 100** est préférée.
* Metric = “coût” → plus petit = priorité.

🎯 Cas concret : bascule de passerelle (redondance).

---

# 🟦 6) Tester les routes avec `ping` et `traceroute`

Sur **vm1** :

```bash
traceroute 8.8.8.8
```

📊 Résultat attendu :

* Si la route par défaut passe par vm2 → premier saut = 10.10.10.12.
* Si elle passe par vm3 → premier saut = 10.10.10.13.

🔎 Explication : traceroute permet de **vérifier quelle route est réellement utilisée**.

🎯 Démo pédagogique : modifie la route par défaut en live, compare les sauts avec traceroute → effet immédiat et visible.

---

# 🟦 7) Routes locales (loopback)

Sur n’importe quelle VM :

```bash
ip route show table local
```

📊 Exemple :

```
local 127.0.0.0/8 dev lo
local 10.10.10.11 dev enp0s8
```

🔎 Explication :

* `local` = adresses propres à la machine (loopback, IPs assignées).
* Ces routes ne quittent jamais la VM.

🎯 Démo utile : montrer que même une IP configurée a **toujours une route locale**.

---

# 🟦 8) Simulation d’une panne de routage

Sur **vm1** :

```bash
sudo ip route del 10.10.10.0/24
```

Puis :

```bash
ping -c2 10.10.10.12
```

📊 Résultat attendu :

```
ping: connect: Network is unreachable
```

🔎 Explication :

* En supprimant la route du LAN, vm1 ne sait plus atteindre ses voisins directs.

🎯 Démo très parlante : même si vm1 est “branchée” (L2 OK), sans route L3 → communication impossible.

Parfait 👍 tu veux garder la **partie 8 (simulation d’une panne de routage)** telle quelle (sans corriger la commande `ip route del 10.10.10.0/24`).
C’est justement intéressant pédagogiquement ✅ car elle permet de **montrer une panne réseau typique** :

---

# 🔎 Partie 8 — Simulation d’une panne de routage

👉 Sur **vm1** :

```bash
sudo ip route del 10.10.10.0/24
```

Ensuite, teste la connectivité vers vm2 :

```bash
ping -c2 10.10.10.12
```

📊 Résultat attendu :

```
ping: connect: Network is unreachable
```

---

### Explication détaillée :

* **Avant** la suppression, vm1 avait cette route :

  ```
  10.10.10.0/24 dev enp0s8 proto kernel scope link src 10.10.10.11
  ```

  👉 Cela dit au noyau : “tout ce qui est dans 10.10.10.0/24 est directement joignable via enp0s8”.

* **Après** la commande `ip route del` :

  * Cette route disparaît.
  * vm1 **ne sait plus comment atteindre son propre LAN** (donc plus de ping vers vm2/vm3).
  * Pourtant la carte réseau est encore UP (`ip link`) et l’IP est toujours présente (`ip addr`).

⚡ **Message clé pour la formation :**

* Niveau **L2** (couche Ethernet) : tout va bien (interface active, MAC connue).
* Niveau **L3** (IP/routage) : la machine n’a **aucune connaissance du réseau 10.10.10.0/24** → impossible d’envoyer les paquets.

---

### Pour rétablir la connectivité :

👉 Toujours sur vm1 :

```bash
sudo ip route add 10.10.10.0/24 dev enp0s8
```

Puis :

```bash
ping -c2 10.10.10.12
# => redevient OK
```

---

🎯 **Intérêt pédagogique dans ton lab 3 VMs :**

* Tu peux faire exécuter `ping` sur vm1, vm2, vm3 → puis supprimer la route sur vm1.
* Les apprenants verront :

  * **vm2 → vm1** = KO (car vm1 ne répond pas).
  * **vm1 → vm2** = KO (`unreachable`).
  * **vm2 ↔ vm3** = OK (ils n’ont pas été modifiés).

