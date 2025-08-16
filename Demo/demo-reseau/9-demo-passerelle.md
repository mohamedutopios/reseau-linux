Super 👍 tu veux maintenant aborder la notion de **passerelle Linux** : transformer une de tes VMs (par ex. **vm2**) en **routeur** entre deux réseaux.

---

# 🌐 1) Qu’est-ce qu’une passerelle Linux ?

* Une **passerelle** (gateway) est une machine qui **fait suivre des paquets d’un réseau à un autre**.
* Par défaut, un Linux **ne route pas** les paquets qu’il reçoit (il ne fait que les traiter pour lui-même).
* Pour qu’il joue le rôle de routeur, il faut **activer le routage IP**.

---

# ⚙️ 2) Activer le routage IP sous Linux

## 🔹 Méthode temporaire (non persistante)

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

ou

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

## 🔹 Méthode persistante (au reboot)

Éditer `/etc/sysctl.conf` et décommenter/ajouter :

```
net.ipv4.ip_forward=1
```

Puis appliquer :

```bash
sudo sysctl -p
```

---

# 🛠️ 3) Exemple dans ton lab (vm2 = passerelle)

Imaginons :

* **vm1 (10.10.10.11)** doit accéder à Internet **via vm2**.
* **vm2 (10.10.10.12)** a deux interfaces :

  * `enp0s8` → LAN privé `10.10.10.0/24`
  * `enp0s3` → NAT (Internet via VirtualBox)

## Étape A – activer le routage IP sur vm2

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

## Étape B – configurer NAT sur vm2 pour masquerader le trafic sortant

```bash
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
```

## Étape C – changer la passerelle par défaut sur vm1

```bash
sudo ip route del default
sudo ip route add default via 10.10.10.12 dev enp0s8
```

## Étape D – test depuis vm1

```bash
ping -c 3 1.1.1.1
curl http://example.com
```

✅ Résultat attendu : vm1 sort vers Internet **via vm2**.

---

# 🧪 4) Exercices pratiques

1. **Sans routage activé** :

   * Configure vm1 pour utiliser vm2 comme passerelle.
   * Test `ping 1.1.1.1` → échoue.
   * Vérifie avec `tcpdump` sur vm2 : paquets arrivent, mais ne repartent pas.

2. **Avec routage activé** :

   * Active `ip_forward` sur vm2.
   * Ajoute règle MASQUERADE.
   * Re-teste depuis vm1 → ça fonctionne.

3. **Blocage sélectif** :

   * Sur vm2, bloque un protocole via `iptables` (ex. ICMP).
   * Depuis vm1 → `ping` échoue, mais `curl` marche.

---

# ✅ Résumé pédagogique

* `/proc/sys/net/ipv4/ip_forward` → active le rôle **routeur** du noyau Linux.
* Sans ça : Linux ne transfère pas les paquets d’une interface à l’autre.
* En l’activant + ajoutant du NAT → une VM devient **passerelle** pour tout un réseau.

---

Veux-tu que je t’écrive un **TP complet "Passerelle Linux"** (énoncé + corrigé) où les apprenants transforment vm2 en routeur/NAT, et où vm1 utilise vm2 comme passerelle pour sortir sur Internet ?
