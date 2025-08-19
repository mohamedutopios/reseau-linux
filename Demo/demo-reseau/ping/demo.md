Super 👍 on attaque maintenant **`ping`** dans ton **lab 3 VMs** :

* **vm1 = 10.10.10.11**
* **vm2 = 10.10.10.12**
* **vm3 = 10.10.10.13**

⚡ `ping` est une commande simple mais **très riche pédagogiquement**. On peut montrer :

* connectivité LAN / WAN,
* résolution DNS,
* MTU et fragmentation,
* latence et stabilité.

---

# 🟦 1) Ping basique LAN (couche 3)

👉 Sur **vm1** :

```bash
ping -c3 10.10.10.12
```

📊 Résultat attendu :

```
64 bytes from 10.10.10.12: icmp_seq=1 ttl=64 time=0.345 ms
64 bytes from 10.10.10.12: icmp_seq=2 ttl=64 time=0.212 ms
64 bytes from 10.10.10.12: icmp_seq=3 ttl=64 time=0.250 ms
```

🔎 Explication :

* `icmp_seq` = numéro de requête ICMP.
* `ttl=64` = nombre de sauts max, ici LAN donc faible.
* `time=0.2-0.3ms` = latence très faible (normal en virtuel).

🎯 Cas d’usage : vérifier rapidement la connectivité vm1 ↔ vm2.

---

# 🟦 2) Ping continu pour surveillance

👉 Sur **vm3** :

```bash
ping 10.10.10.11
```

(Pas de `-c`, ça tourne en continu jusqu’à `Ctrl+C`).

📊 Résultat attendu : flux continu de réponses.

🔎 Explication :

* Utile pour **voir une panne en live** : couper `ip link set enp0s8 down` sur vm1 → le ping vm3 → vm1 se met à afficher `Destination Host Unreachable`.

🎯 Démo très visuelle : montrer aux élèves comment une interface down se reflète en direct.

---

# 🟦 3) Ping vers l’extérieur (connectivité Internet)

👉 Sur **vm1** (si passerelle configurée) :

```bash
ping -c4 8.8.8.8
```

📊 Résultat attendu :

```
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=20 ms
...
```

🔎 Explication :

* Teste si la VM sort bien sur Internet.
* Permet de distinguer problème **LAN** vs problème **WAN**.

🎯 Cas concret : si LAN OK mais `8.8.8.8` KO → problème de route par défaut.

---

# 🟦 4) Ping par nom (test DNS)

👉 Sur **vm1** :

```bash
ping -c3 google.com
```

📊 Résultat attendu :

```
PING google.com (142.250.74.78) 56(84) bytes of data.
```

🔎 Explication :

* Si `ping 8.8.8.8` marche mais pas `ping google.com` → problème DNS.

🎯 Démo : on montre la différence entre **connectivité IP** et **résolution de noms**.

---

# 🟦 5) Ping avec taille personnalisée (tester la MTU)

👉 Sur **vm1** :

```bash
ping -M do -s 1472 10.10.10.12
```

📊 Résultat attendu :

* **OK si MTU=1500**
* **KO si MTU réduite** (ex. `mtu 1400` sur vm2).

🔎 Explication :

* `-s 1472` = 1472 octets de données + 28 octets en-têtes IP+ICMP = 1500.
* `-M do` = “don’t fragment” (ne fragmente pas).

🎯 Démo très claire : montre que si la MTU est trop petite → la communication casse.

---

# 🟦 6) Ping avec intervalle (latence/gigue)

👉 Sur **vm1** :

```bash
ping -i 0.2 -c 10 10.10.10.12
```

📊 Résultat attendu :

* 10 réponses espacées de 0.2s.

🔎 Explication :

* Permet d’observer la **variabilité de la latence** (jitter).
* Utile en téléphonie IP / streaming.

🎯 Démo : comparer latence stable (LAN) vs variable (WAN si tu as une passerelle).

---

# 🟦 7) Ping flood (stress test)

👉 Sur **vm1** :

```bash
sudo ping -f 10.10.10.12
```

📊 Résultat attendu :

* Gros flux `.` (perdus) et `!` (réussis).

🔎 Explication :

* Bombarde de paquets ICMP (attention à ne pas saturer l’hôte physique).
* Sert à tester la charge ou la capacité d’un routeur/firewall.

🎯 Démo encadrée : montrer sur vm2 avec `ip -s link show` → les compteurs RX explosent.

---

# 🟦 8) Simulation de panne de routage (comme avec `ip route`)

👉 Sur **vm1** :

```bash
sudo ip route del 10.10.10.0/24
ping -c3 10.10.10.12
```

📊 Résultat attendu :

```
ping: connect: Network is unreachable
```

🔎 Explication :

* La couche L3 ne sait plus où envoyer les paquets.
* Même si vm2 est branchée, vm1 ne peut pas l’atteindre sans route.

🎯 Démo parfaite : relier `ping` (outil utilisateur) avec la table de routage (`ip route`).

---

✅ Avec ces 8 démos, tu couvres tous les cas d’usage pédagogiques de `ping` :

* LAN vs WAN
* IP vs DNS
* MTU / fragmentation
* Latence / jitter
* Pannes (interface down, route supprimée)
* Stress tests

---

👉 Veux-tu que je prépare ensuite un **TP combiné “ip link + ip addr + ip route + ping”** où les apprenants doivent **dépanner une panne réseau étape par étape** (exercice fil rouge) ?
