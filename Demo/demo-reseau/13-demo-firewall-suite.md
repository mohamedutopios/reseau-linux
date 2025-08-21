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

# Ajouter (à la fin)
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Insérer (au début ou à une position précise)
sudo iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT
sudo iptables -I INPUT 3 -p icmp --icmp-type echo-request -j ACCEPT

# Remplacer la règle n°2 par une autre
sudo iptables -R INPUT 2 -p tcp --dport 443 -j ACCEPT

# Supprimer par n° de ligne
sudo iptables -L INPUT -n -v --line-numbers
sudo iptables -D INPUT 3

# Supprimer par règle “exacte”
sudo iptables -D INPUT -p tcp --dport 80 -j ACCEPT


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

# Ajouter (à la fin)
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Insérer (au début ou à une position précise)
sudo iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT
sudo iptables -I INPUT 3 -p icmp --icmp-type echo-request -j ACCEPT

# Remplacer la règle n°2 par une autre
sudo iptables -R INPUT 2 -p tcp --dport 443 -j ACCEPT

# Supprimer par n° de ligne
sudo iptables -L INPUT -n -v --line-numbers
sudo iptables -D INPUT 3

# Supprimer par règle “exacte”
sudo iptables -D INPUT -p tcp --dport 80 -j ACCEPT

---

# Par n° de ligne (le plus simple)
sudo iptables -L INPUT -n -v --line-numbers
sudo iptables -D INPUT 1   # supprime la règle 1

# Par règle exacte
sudo iptables -D INPUT -p tcp --dport 80 -j ACCEPT

# Reset rapide et sûr (console locale)
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -F && sudo iptables -X


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

Excellent 👍 On va enrichir ta partie **UFW** avec des **démos complètes** adaptées à ton **lab 3 VMs** :

* **vm1 = 10.10.10.11**
* **vm2 = 10.10.10.12**
* **vm3 = 10.10.10.13**
* Interface du lab : **`enp0s8`**

---

# 🚀 2) `ufw` – simplifié (Ubuntu/Debian)

👉 UFW = *Uncomplicated Firewall*, une surcouche simple à iptables.
Très pratique pour un **TP pédagogique**.

---

## 🔹 Étape 0 — Préparation

Installe et réinitialise proprement :

```bash
sudo apt install -y ufw
sudo ufw --force reset
```

Par défaut :

* `deny incoming` (bloque tout en entrée sauf si autorisé)
* `allow outgoing` (autorise tout en sortie)

---

## 🔹 Scénario A — Filtrage **entrant** sur vm2

**Objectif :**

* Autoriser **SSH (22)** seulement depuis vm1
* Bloquer **SSH** depuis vm3
* Autoriser **HTTP (8080)** depuis tout le lab
* Ping : OK depuis vm1, KO depuis vm3

👉 Sur **vm2** :

```bash
# Bloquer tout en entrée, autoriser tout en sortie
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH autorisé uniquement depuis vm1
sudo ufw allow from 10.10.10.11 to any port 22 proto tcp
# SSH bloqué depuis vm3 (sera refusé car règle plus restrictive)
sudo ufw deny from 10.10.10.13 to any port 22 proto tcp

# HTTP 8080 autorisé depuis tout le lab
sudo ufw allow 8080/tcp

# ICMP (ping)
sudo ufw allow from 10.10.10.11 proto icmp
sudo ufw deny  from 10.10.10.13 proto icmp

# Activer
sudo ufw enable
sudo ufw status verbose
```

**Tests** :

* vm1 → vm2:22 ✅
* vm3 → vm2:22 ❌
* vm1/vm3 → vm2:8080 ✅
* vm1 → ping vm2 ✅
* vm3 → ping vm2 ❌

---

## 🔹 Scénario B — Filtrage **sortant** sur vm1

**Objectif :**

* Bloquer les connexions sortantes sur le **port 80** (HTTP)
* Autoriser la sortie vers vm2:8080

👉 Sur **vm1** :

```bash
sudo ufw --force reset
sudo ufw default allow incoming
sudo ufw default deny outgoing

# Autoriser sortie vers vm2:8080
sudo ufw allow out to 10.10.10.12 port 8080 proto tcp

# Bloquer explicitement tout HTTP sortant (port 80)
sudo ufw deny out 80/tcp

sudo ufw enable
sudo ufw status verbose
```

**Tests** :

* `curl -I http://10.10.10.12:8080` → ✅
* `curl -I http://10.10.10.12:80` → ❌

---

## 🔹 Scénario C — Contrôle d’accès précis sur vm3

**Objectif :**

* vm3 écoute sur **9090** (nc -lvnp 9090)
* Autoriser vm2 → vm3:9090
* Bloquer vm1 → vm3:9090

👉 Sur **vm3** :

```bash
sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Autoriser vm2 uniquement
sudo ufw allow from 10.10.10.12 to any port 9090 proto tcp

# Bloquer vm1
sudo ufw deny  from 10.10.10.11 to any port 9090 proto tcp

sudo ufw enable
sudo ufw status verbose
```

**Tests** :

* vm2 → vm3:9090 ✅
* vm1 → vm3:9090 ❌

---

## 🔹 Scénario D — Bloquer/autoriser par interface

Tu peux cibler une interface précise (ici le réseau de lab `enp0s8`).

👉 Sur vm2 (bloquer ping **seulement** sur enp0s8, pas sur l’interface NAT) :

```bash
sudo ufw deny in on enp0s8 proto icmp
```

---

## 🔹 Scénario E — Logging

Activer les logs (utile pour vérifier) :

```bash
sudo ufw logging on
sudo tail -f /var/log/ufw.log
```

Exemple de log quand vm3 tente de ping vm2 alors que c’est bloqué :

```
UFW BLOCK IN=enp0s8 OUT= MAC=... SRC=10.10.10.13 DST=10.10.10.12 LEN=84 TOS=0x00 ...
```

---

## 🔹 Vérifications générales

* Voir le statut :

  ```bash
  sudo ufw status numbered
  ```
* Supprimer une règle par numéro :

  ```bash
  sudo ufw delete 3
  ```
* Réinitialiser complètement :

  ```bash
  sudo ufw --force reset
  ```

---

👉 Avec ça, tu as un TP UFW complet avec **entrées, sorties, adresses IP, ports, interface, logs et suppression**.

Veux-tu que je t’ajoute aussi une **grille d’attendus** (comme pour iptables) avec les tests pour chaque scénario UFW ?
