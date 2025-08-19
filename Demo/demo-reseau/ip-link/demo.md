Parfait 👍 tu veux donc un **TP complet autour de la commande `ip link`**, appliqué à ton lab de **3 VMs** (vm1 = 10.10.10.11, vm2 = 10.10.10.12, vm3 = 10.10.10.13).
Je vais te donner plusieurs **exemples progressifs**, avec :

* 📌 **Commande exacte à lancer**
* 📊 **Résultat attendu / ce qu’on observe**
* 🔎 **Explication claire**
* 🎯 **Lien avec ton lab 3 VMs**

---

# 🟦 1) Lister les interfaces et leurs états

Sur **vm1** :

```bash
ip link
```

📊 Résultat attendu :

```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
```

🔎 Explication :

* `enp0s3` = interface NAT (internet depuis Vagrant/VirtualBox).
* `enp0s8` = interface privée de ton lab (réseau 10.10.10.0/24).
* `<UP,LOWER_UP>` ⇒ interface activée, lien physique OK.

🎯 Dans le lab : toutes tes VMs doivent avoir `enp0s8` en `UP,LOWER_UP` pour communiquer entre elles.

---

# 🟦 2) Statistiques de trafic

Sur **vm1**, avant d’envoyer du trafic :

```bash
ip -s link show enp0s8
```

📊 Résultat attendu :

```
RX: bytes packets errors dropped ...
TX: bytes packets errors dropped ...
```

🔎 Explication : compteurs d’octets/paquets reçus (RX) et envoyés (TX).

🎯 Dans le lab :

* Lance un `ping -c3 10.10.10.12` (vm2 depuis vm1).
* Refais `ip -s link show enp0s8` → tu verras les compteurs **augmenter**.

---

# 🟦 3) Simuler une panne réseau

Sur **vm1** :

```bash
sudo ip link set enp0s8 down
```

Puis teste :

```bash
ping -c2 10.10.10.12
```

📊 Résultat attendu :

```
connect: Network is unreachable
```

Remettre en marche :

```bash
sudo ip link set enp0s8 up
ping -c2 10.10.10.12   # fonctionne à nouveau
```

🔎 Explication :

* `down` = interface administrativement coupée (comme débrancher le câble).
* `up` = interface réactivée.

🎯 Cas concret : très utile pour **montrer aux apprenants une panne réseau** (côté machine et pas switch).

---

# 🟦 4) Changer l’adresse MAC (spoofing)

Sur **vm1** :

```bash
sudo ip link set enp0s8 down
sudo ip link set enp0s8 address 02:11:22:33:44:55
sudo ip link set enp0s8 up
ip link show enp0s8 | grep ether
```

📊 Résultat attendu :

```
link/ether 02:11:22:33:44:55 brd ff:ff:ff:ff:ff:ff
```

🔎 Explication :

* On change l’identité matérielle de la carte.
* Dans le réseau, vm1 apparaît comme une nouvelle machine.
* Si un serveur DHCP existe, il donnera une **nouvelle IP**.

🎯 Dans ton lab : si vm2 est configuré en serveur DHCP, vm1 obtiendra **une IP différente** après ce changement.

---

# 🟦 5) Modifier la MTU (tester la fragmentation)

Sur **vm1** :

```bash
sudo ip link set enp0s8 mtu 1400
ip link show enp0s8 | grep mtu
```

📊 Résultat attendu :

```
3: enp0s8: <...> mtu 1400 qdisc ...
```

🔎 Explication :

* La MTU (Maximum Transmission Unit) limite la taille des paquets.
* Si trop petite → fragmentation ou ping cassé.

🎯 Test concret :

```bash
ping -M do -s 1472 10.10.10.12   # doit échouer (MTU trop basse)
```

Puis remettre la MTU normale :

```bash
sudo ip link set enp0s8 mtu 1500
```

---

# 🟦 6) Voir l’évolution en temps réel

Sur **vm1** :

```bash
watch -n1 'ip -s link show enp0s8'
```

📊 Résultat attendu :

* Les compteurs RX/TX s’incrémentent en direct quand tu pinges ou fais un `curl` vers vm2/vm3.

🎯 Dans le lab :

* Depuis vm1, fais `ping -i0.2 10.10.10.12`.
* Tu verras les compteurs RX/TX augmenter en live.

---

# 🟦 7) Démo “erreurs & collisions” (avancé)

En virtuel (Vagrant/VirtualBox), collisions = toujours 0.
👉 Mais tu peux simuler des **erreurs** en coupant l’interface pendant un transfert.

Sur **vm1** :

```bash
ping 10.10.10.12 > /dev/null &
sleep 2 && sudo ip link set enp0s8 down
```

📊 Résultat attendu :

* Ping cassé.
* `ip -s link show enp0s8` peut montrer des paquets `dropped`.

🎯 Démonstration pédagogique : **effet direct d’une panne réseau** sur les compteurs bas niveau.

---

✅ Avec ces 7 démos, tu couvres **tous les usages de `ip link`** :

* Diagnostic (état, compteurs, erreurs)
* Simulation de panne (down/up)
* Sécurité (spoofing MAC)
* Performance (MTU, collisions, watch en direct)

---

Veux-tu que je t’organise ça sous forme de **TP progressif** (énoncé + corrigé détaillé) pour que tes apprenants puissent le refaire étape par étape dans leur lab ?
