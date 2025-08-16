Très bien 👍 tu veux compléter ton module avec la **Table de routage**.
On va voir :

* Comment **ajouter/supprimer** des routes.
* La différence **routes statiques vs dynamiques**.
* Et des **exercices concrets** dans ton lab (vm1/vm2/vm3).

---

# 📜 1) Qu’est-ce qu’une table de routage ?

* C’est la “carte” utilisée par le noyau pour savoir **par où envoyer les paquets**.
* Chaque entrée = **réseau de destination → interface ou passerelle**.
* Commande de base :

```bash
ip route
```

Exemple attendu dans ton lab (sur vm1) :

```
default via 10.0.2.2 dev enp0s3
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15
10.10.10.0/24 dev enp0s8 proto kernel scope link src 10.10.10.11
```

👉 Ici :

* **default** = passe par l’interface NAT (Internet).
* **10.10.10.0/24** = directement joignable via host-only.

---

# ⚙️ 2) Ajouter / supprimer une route

## ➕ Ajouter une route

* Exemple : sur vm1, ajouter une route fictive vers `192.168.50.0/24` via vm2 (`10.10.10.12`) :

```bash
sudo ip route add 192.168.50.0/24 via 10.10.10.12 dev enp0s8
```

## ➖ Supprimer une route

```bash
sudo ip route del 192.168.50.0/24 via 10.10.10.12 dev enp0s8
```

## 🌍 Changer la **route par défaut**

* Exemple : forcer vm1 à sortir vers Internet via vm2 (au lieu du NAT) :

```bash
sudo ip route del default
sudo ip route add default via 10.10.10.12 dev enp0s8
```

⚠️ Cela ne fonctionnera que si **vm2 fait office de routeur/NAT** → sinon perte d’accès Internet (utile comme test pédagogique).

---

# 🔀 3) Routes statiques vs dynamiques

## 🛑 Routes statiques

* Ajoutées **manuellement** avec `ip route add` ou dans la config réseau.
* Avantages : simple, prévisible.
* Inconvénients : pas d’adaptation en cas de panne/changement.

## ⚡ Routes dynamiques

* Gérées par un **protocole de routage** (ex : RIP, OSPF, BGP) via un démon (`quagga`, `frr`, `bird`).
* Exemple : avec OSPF, les VMs échangeraient automatiquement leurs routes.
* Avantages : adaptation automatique.
* Inconvénients : plus complexe, utile surtout en environnement multi-routeurs.

👉 Dans ton lab actuel, on utilise uniquement des **routes statiques**.
Mais tu pourrais installer **FRRouting (FRR)** pour illustrer les routes dynamiques entre vm2 et vm3.

---

# 🧪 4) Exercices pratiques dans ton lab

### Exercice 1 : Observation

* Sur vm1 :

```bash
ip route show
```

👉 Identifier la route LAN (`10.10.10.0/24`) et la route par défaut (NAT).

### Exercice 2 : Route fictive

* Sur vm1 :

```bash
sudo ip route add 192.168.100.0/24 via 10.10.10.12
ip route show
```

👉 Vérifier que la route apparaît.
Puis la supprimer.

### Exercice 3 : Manipuler la route par défaut

* Sur vm1 : supprimer la route par défaut NAT et mettre vm2 comme passerelle.
* Tester `ping 1.1.1.1`.
  👉 Normalement KO (vm2 n’est pas routeur).
  → Restaurer la route NAT avec :

```bash
sudo ip route add default via 10.0.2.2 dev enp0s3
```

### Exercice 4 : Préparer pour du routage dynamique (optionnel)

* Installer **FRR** sur vm2/vm3 :

```bash
sudo apt-get install frr -y
```

* Configurer OSPF entre vm2/vm3 pour qu’ils échangent des routes automatiquement.
  (utile si tu veux évoluer vers un lab de type “mini réseau”).

---

# ✅ Résumé pédagogique

* **Table de routage** = carte pour savoir où envoyer les paquets.
* **ip route add/del** = manipuler routes statiques.
* **Statique vs dynamique** : manuel (fixe) vs protocole (automatique).
* Dans ton lab :

  * vm1 → client test.
  * vm2 → peut jouer le rôle de “faux routeur”.
  * vm3 → serveur (HTTP).

---

Veux-tu que je prépare un **TP guidé “Routage statique”** (énoncé + corrigé) où les apprenants doivent :

1. ajouter une route fictive,
2. casser/restaurer la route par défaut,
3. comprendre pourquoi Internet ne marche plus si vm2 ne fait pas routeur ?
