Parfait pour un mini-lab L2/L3 local. Avec ces 3 VMs en `10.10.10.0/24` (interfaces host-only) + l’interface NAT par défaut, voici **ce que tu peux faire** avec chaque commande—et des exemples prêts à copier-coller.

# 1) Inspecter et config réseau local

## `ip addr`

* Voir les interfaces et leurs IP (repérer l’host-only, souvent `enp0s8`) :

```bash
ip addr            # toutes les cartes
ip addr show enp0s8
```

* Ajouter une IP secondaire sur l’host-only (test multi-IP) :

```bash
sudo ip addr add 10.10.10.111/24 dev enp0s8
```

* Retirer l’IP secondaire :

```bash
sudo ip addr del 10.10.10.111/24 dev enp0s8
```

## `ip link`

* Voir l’état L2 et les compteurs :

```bash
ip link
ip -s link show enp0s8
```

* Couper/remettre l’interface (observer l’impact sur la connectivité) :

```bash
sudo ip link set enp0s8 down
sudo ip link set enp0s8 up
```

* Tester l’impact d’une MTU différente (utile avec `ping -M do`) :

```bash
sudo ip link set enp0s8 mtu 1400
```

## `ip route`

* Voir la table de routage : la **route par défaut** pointe vers l’interface NAT (`enp0s3`), et le **réseau 10.10.10.0/24** est en direct sur `enp0s8`.

```bash
ip route
```

* Ajouter/Supprimer une route (exercice) :

```bash
# Ex. route "poubelle" pour tester la prise en compte des priorités (metric)
sudo ip route add 8.8.8.8 via 10.10.10.254 metric 50 dev enp0s8
sudo ip route del 8.8.8.8 via 10.10.10.254
```

> Remarque : dans ce lab il n’y a pas de routeur en 10.10.10.254, c’est juste pour manipuler.

# 2) Tester la connectivité IP

## `ping`

* **Inter-VM (L2)** depuis `vm1` :

```bash
ping -c 3 10.10.10.12   # vm2
ping -c 3 10.10.10.13   # vm3
```

* **Vers Internet (via NAT)** :

```bash
ping -c 3 1.1.1.1
```

* **Taille MTU / fragmentation** :

```bash
ping -M do -s 1472 1.1.1.1   # 1472+28=1500, varie si tu as changé la MTU
```

## `traceroute`

* **Inter-VM** (même LAN) → une seule “saut” (direct) attendu :

```bash
sudo apt-get update && sudo apt-get install -y traceroute   # si absent
traceroute 10.10.10.12
```

* **Vers Internet** (montre le NAT) :

```bash
traceroute 8.8.8.8
```

# 3) Résolution DNS (noms ↔ IP)

Par défaut, tes VMs utiliseront le résolveur fourni par l’interface NAT (VirtualBox). Tu peux :

* **Résoudre des noms publics** :

```bash
nslookup www.example.com
dig www.example.com
dig +short www.example.com
```

* **Spécifier un serveur DNS** :

```bash
nslookup www.example.com 1.1.1.1
dig @8.8.8.8 www.debian.org
```

* **Reverse lookup** (PTR) :

```bash
dig -x 8.8.8.8 +short
```

* **Noms locaux entre VMs** (il n’y a pas de DNS local) :

  * Option 1 : utiliser les IP directement.
  * Option 2 : ajouter des entrées dans `/etc/hosts` sur **chaque** VM :

    ```
    10.10.10.11 vm1
    10.10.10.12 vm2
    10.10.10.13 vm3
    ```

    Puis :

    ```bash
    ping -c 3 vm2
    nslookup vm3
    ```

# 4) Petits labs rapides

1. **Inventaire réseau** : sur chaque VM, collecte `ip addr`, `ip link`, `ip route` et explique le rôle de chaque interface.
2. **Coupure L2** : mets `enp0s8` down sur `vm2` puis `ping` depuis `vm1` → observe l’échec, remets up → succès.
3. **MTU & ping** : change la MTU à 1400 sur `vm3`, teste `ping -M do -s 1472` vers un IP public → compare.
4. **DNS public vs local** : crée des entrées `/etc/hosts` pour `vm1/vm2/vm3`, vérifie `nslookup vm2` (réponse “non-authoritative” locale) puis `dig @1.1.1.1 vm2` (échec attendu, pas de DNS pour ce nom).
5. **Traceroute** : compare `traceroute 10.10.10.12` (LAN direct) vs `traceroute 9.9.9.9` (plusieurs sauts via NAT/FAI).

# 5) Paquets utiles (si manquants)

```bash
sudo apt-get update
sudo apt-get install -y iproute2 inetutils-ping traceroute dnsutils
```

# 6) Attendus & dépannage

* **Inter-VM** : ping OK entre `10.10.10.11/12/13`. Si KO → vérifier `ip addr` (bonne IP/masque), `ip link` (UP), firewall (`ufw status`).
* **Internet** : ping IP public OK via NAT ; si KO → vérifier la route par défaut (`ip route | grep default`) et que l’interface NAT (`enp0s3`) est UP.
* **DNS** : si `nslookup` lent/KO, forcer un DNS connu (`dig @1.1.1.1 example.com`) et vérifier `/etc/resolv.conf`.

Si tu veux, je te propose ensuite un mini-TP pour monter un **DNS léger (dnsmasq)** sur `vm1` et résoudre `vm1.lab`, `vm2.lab`, `vm3.lab` pour professionnaliser les tests `nslookup/dig`.
