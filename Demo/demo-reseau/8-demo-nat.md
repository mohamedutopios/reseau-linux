Parfait 👍 tu veux maintenant attaquer la partie **NAT et masquerading** dans ton lab.
C’est une bonne suite après la table de routage, car NAT = “traduction d’adresses” pour que plusieurs machines privées puissent sortir via une seule IP publique.

---

# 🌍 1) Qu’est-ce que le NAT / Masquerading ?

* **NAT (Network Address Translation)** : traduit les adresses IP source/destination lors du passage d’un paquet.

  * Exemple : `10.10.10.11` (privée) → `192.0.2.100` (publique).
* **Masquerading** : type particulier de NAT source où l’adresse de sortie est **prise automatiquement sur l’interface sortante** (utile pour des IP dynamiques, comme une box Internet).
* Objectif :

  * Donner accès à Internet à des machines privées.
  * Faire du port forwarding (accéder à un service interne depuis l’extérieur).

---

# ⚙️ 2) Où configurer le NAT dans Linux ?

* NAT utilise la table spéciale `nat` d’**iptables**.
* Trois chaînes principales :

  * **PREROUTING** → changer la destination (DNAT).
  * **POSTROUTING** → changer la source (SNAT/masquerade).
  * **OUTPUT** → pour les paquets générés localement.

👉 Exemple classique : **masquerading sur l’interface vers Internet** (`enp0s3` dans ton lab, l’interface NAT).

---

# 🛠️ 3) Exemple concret dans ton lab

Scénario :

* **vm2** joue le rôle de “routeur/NAT”.
* **vm1** et **vm3** passent par vm2 pour accéder à Internet.

## Étape A : activer le routage IP sur vm2

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

(ajouter `net.ipv4.ip_forward=1` dans `/etc/sysctl.conf` pour persister)

## Étape B : ajouter une règle de masquerading

👉 Sur vm2 (qui a accès Internet via enp0s3/NAT) :

```bash
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
```

## Étape C : configurer vm1 et vm3 pour utiliser vm2 comme passerelle par défaut

👉 Sur vm1 et vm3 :

```bash
sudo ip route del default
sudo ip route add default via 10.10.10.12 dev enp0s8
```

## Étape D : tester

👉 Depuis vm1 :

```bash
ping -c 3 1.1.1.1
curl http://example.com
```

✅ Ça passe par vm2 → NAT → Internet.

---

# 🔀 4) Différence entre SNAT et MASQUERADE

* **SNAT (Source NAT)** : on indique explicitement l’adresse source de sortie.
  Exemple :

```bash
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j SNAT --to-source 192.0.2.100
```

* **MASQUERADE** : l’adresse de sortie est prise automatiquement (utile quand l’IP change, ex. DHCP, box).

```bash
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
```

👉 Dans un lab Vagrant/VirtualBox (IP NAT dynamique fournie), **MASQUERADE est recommandé**.

---

# 🧪 5) Exercices pratiques

### Exercice 1 : Sans NAT

* Configure vm1 pour passer par vm2 (`ip route add default via 10.10.10.12`).
* Sans masquerading sur vm2 → `ping 1.1.1.1` échoue.
  👉 Montre que vm2 reçoit bien le paquet mais ne peut pas le renvoyer.

### Exercice 2 : Avec NAT

* Active `iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE` sur vm2.
* Depuis vm1 → `ping 1.1.1.1` fonctionne.
  👉 Montre la différence.

### Exercice 3 : DNAT (port forwarding)

* Sur vm2, rediriger le port 8080 vers Apache sur vm3 (80) :

```bash
sudo iptables -t nat -A PREROUTING -i enp0s3 -p tcp --dport 8080 -j DNAT --to-destination 10.10.10.13:80
```

* Depuis l’hôte (ton PC), accéder à `http://127.0.0.1:8080` → doit arriver sur Apache de vm3.

---

# ✅ Résumé pédagogique

* **iptables -t nat** = gérer NAT et masquerading.
* **Masquerade** = NAT source automatique (pratique pour IP dynamiques).
* **Scénario typique** : vm2 en routeur NAT → vm1/vm3 sortent via vm2.
* **DNAT** = port forwarding → accès externe vers un service interne.

---

Veux-tu que je te prépare un **TP guidé “NAT & masquerading”** (énoncé + corrigé) où les apprenants :

1. cassent la sortie Internet en changeant la gateway,
2. rétablissent avec iptables + MASQUERADE,
3. ajoutent un DNAT pour exposer Apache de vm3 via vm2 ?
