Parfait 👍 tu veux maintenant compléter ton lab avec une **introduction au pare-feu**.
On va voir une **présentation claire** de deux outils disponibles dans Ubuntu : **iptables** (bas niveau, puissant) et **ufw** (frontend simplifié).

---

# 🔥 1) Qu’est-ce qu’un pare-feu Linux ?

* Le noyau Linux intègre un module de filtrage réseau (netfilter).
* Les paquets réseau passent par des **tables** et **chaînes** qui définissent des règles (ACCEPT, DROP, REJECT).
* On peut ainsi :

  * Autoriser certains flux.
  * Bloquer d’autres.
  * Faire du NAT, du port forwarding, de la journalisation, etc.

---

# ⚙️ 2) `iptables` – le classique bas niveau

👉 **Concepts clés** :

* **Tables** : filter (par défaut), nat, mangle, raw.
* **Chaînes** :

  * INPUT → trafic entrant vers la machine.
  * OUTPUT → trafic sortant.
  * FORWARD → trafic routé à travers la machine.
* **Actions** : ACCEPT, DROP, REJECT, LOG…

👉 **Exemples pratiques** :

* Lister les règles actuelles :

```bash
sudo iptables -L -n -v
```

* Bloquer tout le trafic ICMP (ping) entrant :

```bash
sudo iptables -A INPUT -p icmp -j DROP
```

* Autoriser SSH uniquement depuis vm1 (10.10.10.11) :

```bash
sudo iptables -A INPUT -p tcp --dport 22 -s 10.10.10.11 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j DROP
```

* Supprimer toutes les règles (remise à zéro) :

```bash
sudo iptables -F
```

👉 **Avantage** : très puissant, granulaire.
👉 **Inconvénient** : syntaxe complexe, peu lisible pour débutants.

---

# 🚀 3) `ufw` – uncomplicated firewall

👉 **C’est un frontend d’iptables**, simplifié, disponible sur Ubuntu.
👉 Idéal pour un usage basique (autoriser/refuser quelques ports).

* Activer ufw :

```bash
sudo ufw enable
```

* Autoriser SSH :

```bash
sudo ufw allow ssh
```

* Autoriser HTTP :

```bash
sudo ufw allow 80/tcp
```

* Bloquer un service :

```bash
sudo ufw deny 5000/udp
```

* Voir l’état :

```bash
sudo ufw status verbose
```

* Désactiver :

```bash
sudo ufw disable
```

👉 **Avantage** : simple, clair.
👉 **Inconvénient** : pas toutes les fonctionnalités avancées d’iptables.

---

# 🧪 4) Exercices adaptés à ton lab

### 🔑 SSH

* Sur **vm2**, autorise SSH uniquement depuis vm1.
* Depuis vm3 → tentative SSH doit échouer.

### 🌐 Web (Apache sur vm3)

* Active ufw et autorise uniquement le port 80.
* Test depuis vm1 : `curl http://10.10.10.13` → OK.
* Test `ssh vm3` → doit échouer.

### 📡 UDP 5000 (vm2)

* Lance `nc -lu 5000` sur vm2.
* Bloque le port avec ufw :

  ```bash
  sudo ufw deny 5000/udp
  ```
* Test depuis vm1 :

  ```bash
  echo "test" | nc -u 10.10.10.12 5000
  ```

  → bloqué.

---

# ✅ Résumé pédagogique

* **iptables** = outil bas niveau, granulaire, utilisé par tous les systèmes Linux.
* **ufw** = surcouche simplifiée pour Ubuntu/Debian, adaptée aux labs.
* Dans ton lab :

  * **vm2** → tester SSH + UDP firewall.
  * **vm3** → tester HTTP firewall.
  * **vm1** → sert de client pour tester.

---

Veux-tu que je prépare un **TP guidé “Pare-feu”** (énoncé + corrigé) avec une progression :

1. inspection sans règles,
2. blocage ICMP,
3. filtrage SSH,
4. autorisation HTTP,
5. blocage UDP ?
