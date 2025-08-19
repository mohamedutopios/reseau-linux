Parfait 👌
On reste dans ton **lab 3 VMs (vm1 = 10.10.10.11, vm2 = 10.10.10.12, vm3 = 10.10.10.13)** et cette fois on se concentre sur **`ip addr`** (gestion des adresses IP, couche 3).

Chaque démo =

* 📌 **Commande exacte**
* 📊 **Résultat attendu**
* 🔎 **Explication claire**
* 🎯 **Lien avec ton lab**

---

# 🟦 1) Afficher toutes les IP

Sur **vm1** :

```bash
ip addr
```

📊 Exemple de sortie :

```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
    inet 10.10.10.11/24 brd 10.10.10.255 scope global enp0s8
```

🔎 Explication :

* `inet 10.10.10.11/24` = IP + masque (vm1 sur ton LAN privé).
* `brd` = adresse broadcast du réseau.
* `scope global` = utilisable pour communiquer en dehors.

🎯 Contexte lab : permet de vérifier rapidement que **chaque VM a bien son IP dans 10.10.10.0/24**.

---

# 🟦 2) Voir uniquement une interface

Sur **vm2** :

```bash
ip addr show enp0s8
```

📊 Exemple :

```
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
    inet 10.10.10.12/24 brd 10.10.10.255 scope global enp0s8
```

🔎 Explication :

* On filtre uniquement `enp0s8`.
* Vérification rapide en dépannage (si interface NAT/Internet embrouille la sortie).

🎯 Utile si un apprenant confond l’IP “Internet” (NAT) et l’IP du LAN privé.

---

# 🟦 3) Ajouter une IP secondaire (multi-IP)

Sur **vm2** :

```bash
sudo ip addr add 10.10.10.222/24 dev enp0s8
ip addr show enp0s8 | grep inet
```

📊 Résultat attendu :

```
inet 10.10.10.12/24 scope global enp0s8
inet 10.10.10.222/24 scope global secondary enp0s8
```

🔎 Explication :

* L’interface peut avoir **plusieurs IPs en même temps**.
* Le trafic vers `.12` ou `.222` ira bien vers vm2.

🎯 Dans ton lab : depuis **vm1** fais :

```bash
ping -c2 10.10.10.222
```

→ Tu touches la même machine par une **deuxième identité IP**.

---

# 🟦 4) Supprimer une IP spécifique

Sur **vm2** :

```bash
sudo ip addr del 10.10.10.222/24 dev enp0s8
ip addr show enp0s8
```

📊 Résultat attendu :

```
inet 10.10.10.12/24 scope global enp0s8
```

(l’IP secondaire n’apparaît plus)

🔎 Explication :

* Retire uniquement l’IP indiquée, sans toucher aux autres.
* Contraire de `ip addr add`.

🎯 Cas réel : nettoyage d’une IP temporaire ajoutée pour un test.

---

# 🟦 5) Changer complètement l’IP

Sur **vm3** :

```bash
sudo ip addr flush dev enp0s8
sudo ip addr add 10.10.10.99/24 dev enp0s8
ip addr show enp0s8
```

📊 Résultat attendu :

```
inet 10.10.10.99/24 scope global enp0s8
```

🔎 Explication :

* `flush` efface toutes les IP existantes.
* `add` définit une nouvelle IP.
* vm3 est maintenant `.99` au lieu de `.13`.

🎯 Dans ton lab :

* Depuis vm1, tester `ping 10.10.10.99` fonctionne.
* `ping 10.10.10.13` échoue (ancienne IP n’existe plus).

---

# 🟦 6) Ajouter une IP dans un autre réseau (multi-homing)

Sur **vm1** :

```bash
sudo ip addr add 192.168.50.10/24 dev enp0s8
ip addr show enp0s8 | grep inet
```

📊 Résultat attendu :

```
inet 10.10.10.11/24 scope global enp0s8
inet 192.168.50.10/24 scope global secondary enp0s8
```

🔎 Explication :

* vm1 appartient maintenant **à deux réseaux logiques différents**.
* Elle peut communiquer avec des hôtes de `10.10.10.0/24` **et** `192.168.50.0/24` (si connectés).

🎯 Cas concret : simuler un serveur connecté à deux sous-réseaux (ex. DMZ + LAN interne).

---

# 🟦 7) Vérifier l’adresse obtenue par DHCP

Sur **vm1** (si vm2 est DHCP server) :

```bash
sudo dhclient -v -r enp0s8   # libère l’IP
sudo dhclient -v enp0s8      # redemande une IP
ip addr show enp0s8 | grep inet
```

📊 Résultat attendu :

```
inet 10.10.10.101/24 scope global dynamic enp0s8
```

🔎 Explication :

* L’option `dynamic` apparaît si l’IP vient d’un serveur DHCP.
* On peut voir le bail changer selon la config du serveur.

🎯 Dans ton lab : vm1 et vm3 peuvent obtenir dynamiquement des IPs si vm2 fait serveur DHCP.

---

# 🟦 8) Jouer avec les labels (nommer une IP secondaire)

Sur **vm2** :

```bash
sudo ip addr add 10.10.10.223/24 dev enp0s8 label enp0s8:1
ip addr show enp0s8
```

📊 Résultat attendu :

```
inet 10.10.10.223/24 scope global secondary enp0s8:1
```

🔎 Explication :

* On attribue un alias (`enp0s8:1`) à l’IP.
* Historiquement utilisé pour gérer plusieurs IPs facilement.

🎯 Démo utile pour montrer aux apprenants l’idée des “alias d’interface”.

---

✅ Avec ces 8 démos, tu couvres tout le spectre de `ip addr` :

* Affichage simple (`show`)
* Multi-IP (`add`, `del`)
* Reconfiguration totale (`flush`, `add`)
* Multi-réseaux (multi-homing)
* DHCP (`dhclient`)
* Aliases (`label`)

---

Veux-tu que je fasse la même chose pour **`ip route`** (routage), pour que tu aies la trilogie complète `ip link / ip addr / ip route` bien documentée pour ton cours ?
