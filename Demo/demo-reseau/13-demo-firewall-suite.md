Parfait 👍 tu veux maintenant une partie **Pare-feu avancé** dans ton lab.
On va voir :

1. **Filtrage entrant/sortant avec `iptables`**
2. **Utilisation simplifiée avec `ufw` (Ubuntu) ou `firewalld` (plus courant en RedHat/CentOS)**
3. **Exercices pratiques sur tes VM (vm1, vm2, vm3)**

---

# 🔥 1) `iptables` – filtrage avancé

👉 **Rappel rapide**

* Table `filter` (par défaut) : chaînes **INPUT**, **OUTPUT**, **FORWARD**
* Politique par défaut : `ACCEPT` (tout passe).
* Action : `ACCEPT`, `DROP`, `REJECT`, `LOG`.

---

## ✅ Exemple : filtrage entrant

* Bloquer tout le trafic entrant sauf SSH (22) et HTTP (80) :

```bash
sudo iptables -P INPUT DROP
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

---

## ✅ Exemple : filtrage sortant

* Autoriser uniquement HTTP/HTTPS sortants, bloquer tout le reste :

```bash
sudo iptables -P OUTPUT DROP
sudo iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT
```

---

## ✅ Exemple : blocage ICMP (ping)

```bash
sudo iptables -A INPUT -p icmp -j DROP
```

---

## ✅ Voir les règles

```bash
sudo iptables -L -n -v
```

---

# 🚀 2) `ufw` – simplifié (Ubuntu/Debian)

👉 UFW = *Uncomplicated Firewall* (surcouche à iptables).
Très utile pour un usage rapide dans ton lab Ubuntu.

### Exemples :

* Activer ufw :

```bash
sudo ufw enable
```

* Autoriser SSH et HTTP :

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
```

* Bloquer tout le reste :

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

* Vérifier :

```bash
sudo ufw status verbose
```

---

# 🔧 3) `firewalld` – zones et services (RedHat/CentOS/Fedora)

👉 `firewalld` est une alternative moderne, basée sur `zones`.
Pas installé par défaut sur Ubuntu, mais utile à connaître.

* Installer (Ubuntu) :

```bash
sudo apt install -y firewalld
```

* Exemples :

```bash
sudo firewall-cmd --get-zones
sudo firewall-cmd --zone=public --add-service=ssh --permanent
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --reload
```

* Vérifier :

```bash
sudo firewall-cmd --list-all
```

---

# 🧪 4) Exercices pratiques pour ton lab

👉 **vm2 = passerelle / serveur**
👉 **vm3 = serveur web**
👉 **vm1 = client/test**

### Exercice 1 : Bloquer ICMP

* Sur vm2, bloque ICMP avec iptables.
* Depuis vm1, `ping vm2` doit échouer, mais SSH doit marcher.

### Exercice 2 : Autoriser uniquement SSH et HTTP

* Sur vm3 (serveur web), configure ufw pour n’autoriser que `22` et `80`.
* Depuis vm1 :

  * `ssh vm3` → OK
  * `curl http://vm3` → OK
  * `nc -zv vm3 5000` → bloqué

### Exercice 3 : Bloquer la sortie Internet

* Sur vm1, bloque tout le trafic sortant sauf DNS (53) et HTTP/HTTPS.
* Teste avec :

  ```bash
  ping 1.1.1.1      # KO
  dig google.com    # OK
  curl http://example.com  # OK
  ```

### Exercice 4 (optionnel) : firewalld

* Installe `firewalld` sur vm2, configure la zone `public` pour n’autoriser que SSH et HTTP.
* Vérifie avec `firewall-cmd --list-all`.

---

# ✅ Résumé pédagogique

* **iptables** = flexible, puissant, granulaire → bien pour comprendre.
* **ufw** = rapide et simple sur Ubuntu → idéal pour ton lab.
* **firewalld** = plus utilisé côté RedHat, basé sur des zones → bon à connaître.

---

Veux-tu que je rédige un **TP complet “Pare-feu avancé”** (énoncé + corrigé) qui mélange iptables, ufw et un peu de firewalld, avec des scénarios concrets dans ton lab ?
