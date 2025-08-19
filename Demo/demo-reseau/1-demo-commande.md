Parfait 👍 je comprends ce que tu veux : pas seulement une liste de commandes mais aussi **leurs utilisations variées**, dans plusieurs contextes concrets.
Je vais donc te proposer un guide **structuré par commande**, avec :

* 📝 **Explication générale**
* 🔧 **Exemples dans ton lab (3 VMs)**
* 🎯 **Utilisations variées (dépannage, sécurité, tests avancés, cas réels)**

---

# 🔹 1) `ip link` (couche 2 – interface physique)

### Explication

Permet de gérer l’état de l’interface, ses paramètres bas-niveau (MAC, MTU, état UP/DOWN).

### Exemples de base

```bash
ip link                # liste toutes les interfaces
ip -s link show enp0s8 # stats sur enp0s8
```
Voici comment lire cette sortie pour `enp0s8` (ton interface privée Vagrant/VirtualBox) :

**Ligne 1**

* `3:` index de l’interface dans le noyau.
* `enp0s8` nom de l’interface.
* `<BROADCAST,MULTICAST,UP,LOWER_UP>` **flags** :

  * `BROADCAST` supporte les trames broadcast.
  * `MULTICAST` supporte le multicast (ARP/MDNS/…).
  * `UP` l’interface est activée (administrativement up).
  * `LOWER_UP` le lien physique est actif (carrier OK).
* `mtu 1500` taille max d’une trame IP.
* `qdisc fq_codel` algorithme de file d’attente (Fair Queuing Controlled Delay), bon contre la bufferbloat.
* `state UP` état opérationnel.
* `mode DEFAULT` mode normal (pas monitor).
* `group default` groupe de l’interface.
* `qlen 1000` longueur de file TX (1000 paquets max en attente).

**Ligne 2**

* `link/ether 08:00:27:ac:0a:69` MAC de l’interface.
* `brd ff:ff:ff:ff:ff:ff` adresse MAC de broadcast.

**Compteurs RX (réception)**

```
RX:  bytes  packets  errors  dropped  missed  mcast
       3146       23       0        0       0      1
```

* `bytes/packets` trafic reçu (3 146 octets / 23 paquets).
* `errors` erreurs de réception (0 ⇒ RAS).
* `dropped` paquets rejetés par la pile réseau/file (0).
* `missed` paquets manqués par le driver (souvent 0).
* `mcast` paquets multicast reçus (ici 1).

**Compteurs TX (émission)**

```
TX:  bytes  packets  errors  dropped  carrier  collsns
       1306      17       0        0       0        0
```

* `bytes/packets` émis (1 306 octets / 17 paquets).
* `errors` erreurs d’émission (0).
* `dropped` paquets rejetés avant émission (0).
* `carrier` erreurs de porteuse (0; typique en virtuel).
* `collsns` collisions (0; normal en switché/virtuel).

### À retenir / interprétation rapide

* L’interface est **UP** et le lien **physique OK** (`LOWER_UP`).
* Trafic très léger (quelques ARP/ICMP/DNS, etc.).
* **Aucune** erreur ni drop ⇒ santé réseau OK.
* `fq_codel` est l’ordonnanceur par défaut moderne.

### Astuces utiles

* Voir l’évolution en direct :
  `watch -n1 'ip -s link show enp0s8'`
* Remettre les compteurs à zéro (sans reboot) :
  `ip link set dev enp0s8 down && ip link set dev enp0s8 up`
* Détails driver/NIC : `ethtool -S enp0s8` (installer `ethtool`).
* Voir la QoS exacte : `tc qdisc show dev enp0s8`.
* Tester rapidement : `ping -c3 10.10.10.12` (ou une autre VM) puis relis `ip -s link` pour voir les compteurs bouger.

### Utilisations variées

* **Couper/activer une interface** (simuler panne câble ou port switch) :

  ```bash
  sudo ip link set enp0s8 down
  sudo ip link set enp0s8 up
  ```
* **Changer la MTU** (tester tunnels VPN, fragmentation) :

  ```bash
  sudo ip link set enp0s8 mtu 1400
  ```
* **Changer d’adresse MAC (spoofing)** (utile en sécurité/pentest) :

  ```bash
  sudo ip link set enp0s8 address 02:11:22:33:44:55
  ```
* **Vérifier collisions/erreurs** avec les compteurs (`ip -s link`) → diagnostic de problèmes physiques.

---

# 🔹 2) `ip addr` (couche 3 – adresses IP)

### Explication

Affiche, ajoute, supprime des adresses IP sur les interfaces.

### Exemples de base

```bash
ip addr show enp0s8
```

### Utilisations variées

* **Ajouter une IP secondaire (multi-homing)** :

  ```bash
  sudo ip addr add 10.10.10.111/24 dev enp0s8
  ```

  → Simule une machine qui joue 2 rôles (par ex. serveur web interne + externe).
* **Changer l’adresse en live** (sans redémarrer) :

  ```bash
  sudo ip addr flush dev enp0s8
  sudo ip addr add 192.168.1.10/24 dev enp0s8
  ```
* **Analyser une configuration DHCP** (la commande montre quelle IP/mask le DHCP a donné).
* **Associer plusieurs services** (un serveur peut avoir une IP différente pour chaque appli).

---

# 🔹 3) `ip route` (routage IP)

### Explication

Affiche et modifie la table de routage, essentielle pour savoir **où vont les paquets**.

### Exemples de base

```bash
ip route
```

### Utilisations variées

* **Ajouter une route statique** :

  ```bash
  sudo ip route add 192.168.50.0/24 via 10.10.10.12
  ```

  → Simule un routeur VM2 qui connaît le chemin vers un autre réseau.
* **Changer la route par défaut** (forcer à passer par une autre passerelle) :

  ```bash
  sudo ip route change default via 10.10.10.254
  ```
* **Tester métriques de routage (priorités)** :

  ```bash
  sudo ip route add 8.8.8.8 via 10.10.10.12 metric 100
  ```
* **Dépannage** : si tu pinges Internet mais que ça échoue → vérifier que `default via …` existe.

---

# 🔹 4) `ping` (connectivité réseau)

### Explication

Teste si une machine est accessible (ICMP Echo).

### Exemples de base

```bash
ping -c 3 10.10.10.12
```

### Utilisations variées

* **Test LAN vs Internet** :

  ```bash
  ping 10.10.10.12    # local
  ping 8.8.8.8        # Internet
  ```
* **Test DNS vs IP** (diagnostic résolution) :

  ```bash
  ping google.com
  ```
* **Tester la taille max sans fragmentation (MTU)** :

  ```bash
  ping -M do -s 1472 1.1.1.1
  ```
* **Mesurer latence** (utile en perf réseau) :

  ```bash
  ping -i 0.2 -c 10 8.8.8.8
  ```

---

# 🔹 5) `traceroute` (chemin des paquets)

### Explication

Montre le chemin (nombre de sauts) qu’un paquet prend.

### Exemples de base

```bash
traceroute 8.8.8.8
```

### Utilisations variées

* **Comparer LAN vs WAN** :

  ```bash
  traceroute 10.10.10.12   # 1 seul saut
  traceroute 8.8.8.8       # plusieurs sauts
  ```
* **Identifier un blocage** (où les paquets s’arrêtent).
* **Analyser un routage asymétrique** (si certains chemins passent par NAT, d’autres non).
* **Diagnostic perf** : latence élevée sur un saut particulier → problème chez l’opérateur.

---

# 🔹 6) `dig` / `nslookup` (résolution DNS)

### Explication

Teste la résolution de noms en adresses IP.

### Exemples de base

```bash
dig www.example.com
nslookup www.google.com
```

### Utilisations variées

* **Tester le DNS configuré** (par défaut NAT VirtualBox) :

  ```bash
  dig www.debian.org
  ```
* **Spécifier un DNS** (Cloudflare, Google) :

  ```bash
  dig @1.1.1.1 www.example.com
  ```
* **Vérifier reverse DNS** :

  ```bash
  dig -x 8.8.8.8 +short
  ```
* **Créer un petit DNS local** (via `/etc/hosts` ou dnsmasq) → `ping vm2.lab` fonctionne.

---

# 🔹 7) Commandes complémentaires utiles

* **`arp` / `ip neigh`** : voir les adresses MAC connues.

  ```bash
  ip neigh show
  ```

  → Diagnostic si VM1 connaît l’adresse MAC de VM2.

* **`ss` / `netstat`** : voir les connexions ouvertes.

  ```bash
  ss -tulnp
  ```

  → Utile pour vérifier si un service écoute bien.

* **`curl` / `wget`** : tester HTTP.

  ```bash
  curl http://10.10.10.12:8080
  ```

---

✅ Avec ça, tu as non seulement les **commandes de base**, mais aussi **leurs utilisations réelles** : dépannage, labo, sécurité, optimisation.
Chaque outil a une **utilité précise** pour comprendre et agir sur ton réseau.

👉 Veux-tu que je te prépare un **TP structuré** avec des scénarios (ex : "VM1 n’arrive pas à joindre Internet, trouver pourquoi en utilisant ces commandes") pour mettre tout ça en pratique comme un vrai examen ?
