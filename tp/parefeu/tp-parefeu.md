Parfait — voici le **TP pare-feu complet** (tests inclus) pour ton lab **3 VMs Ubuntu**.
On te donne **2 versions** (fais **l’une OU l’autre**, pas les deux en même temps) :

* Partie A : **UFW** (pédago et rapide)
* Partie B : **iptables** (équivalent bas niveau)

Topologie (à adapter si besoin) :

* **vm1** = `10.10.10.11`
* **vm2** = `10.10.10.12`
* **vm3** = `10.10.10.13`
* Interface du lab : **`enp0s8`** (ne filtre que là-dessus pour ne pas casser l’accès NAT/Internet)

---

# Préparation commune (sur **chaque VM**)

```bash
# Outils utiles
sudo apt update
sudo apt install -y curl netcat-openbsd tcpdump iproute2

# (Optionnel) nmap pour vérifier les ports
# sudo apt install -y nmap
```

# Services de test

## vm2 (HTTP sur 8080)

```bash
# Terminal 1 (interactif) :
cd /tmp && python3 -m http.server 8080
# ou en arrière-plan si tu préfères :
# nohup python3 -m http.server 8080 >/tmp/http8080.log 2>&1 &
```

## vm3 (TCP sur 9090)

```bash
# Terminal 2 (interactif et persistant) :
nc -lvk -p 9090
# ou en arrière-plan :
# nohup nc -lvk -p 9090 >/tmp/nc9090.log 2>&1 &
```

## Vérifier l’écoute (sur vm2 et vm3)

```bash
sudo ss -ltnp | egrep ':8080|:9090'
```

## Tests de base (depuis vm1 et vm3)

```bash
ping -c2 10.10.10.12
curl -I http://10.10.10.12:8080
nc -zv -w 3 10.10.10.13 9090
```

> Attends-toi à tout passer **OK** avant d’activer le pare-feu.

---

# 🔵 Partie A — TP avec **UFW** (choix recommandé pour débuter)

⚠️ Fais **UNIQUEMENT** cette partie si tu choisis UFW.
(UFW pilote iptables en arrière-plan ; n’enchaîne pas avec la partie iptables sans reset.)

## Scénario A — **Filtrage entrant** sur **vm2**

Objectifs :

* **SSH (22/TCP)** : autorisé **seulement** depuis vm1 ; bloqué depuis vm3.
* **HTTP 8080/TCP** : autorisé depuis tout le lab.
* **ICMP (ping)** : autorisé depuis vm1 ; **bloqué** depuis vm3.

Sur **vm2** :

```bash
sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Autoriser SSH de vm1, bloquer (et log) vm3
sudo ufw allow in on enp0s8 from 10.10.10.11 to any port 22 proto tcp
sudo ufw deny  log in on enp0s8 from 10.10.10.13 to any port 22 proto tcp

# HTTP 8080 pour tout le lab
sudo ufw allow in on enp0s8 to any port 8080 proto tcp

# ICMP: OK pour vm1, KO pour vm3
sudo ufw allow in on enp0s8 from 10.10.10.11 to any proto icmp
sudo ufw deny  in on enp0s8 from 10.10.10.13 to any proto icmp

# Logs + activation
sudo ufw logging on
sudo ufw enable
sudo ufw status numbered
```

**Tests**

* Depuis **vm1** :
  `ssh vagrant@10.10.10.12` → **OK**
  `ping -c2 10.10.10.12` → **OK**
  `curl -I http://10.10.10.12:8080` → **200/301/404**
* Depuis **vm3** :
  `ssh vagrant@10.10.10.12` → **REFUSÉ**
  `ping -c2 10.10.10.12` → **REFUSÉ**
  `curl -I http://10.10.10.12:8080` → **OK**
* Logs : `sudo tail -f /var/log/ufw.log` (sur vm2)

---

## Scénario B — **Filtrage sortant** sur **vm1**

Objectifs :

* **Bloquer** la **sortie 80/TCP** sur l’interface lab.
* **Autoriser** la **sortie vers vm2:8080/TCP**.

Sur **vm1** :

```bash
sudo ufw --force reset
sudo ufw default allow outgoing
sudo ufw default allow incoming

# Autoriser la sortie vers vm2:8080 en priorité
sudo ufw allow out on enp0s8 to 10.10.10.12 port 8080 proto tcp

# Bloquer la sortie HTTP (80/TCP) sur le lab
sudo ufw deny out on enp0s8 to any port 80 proto tcp

sudo ufw enable
sudo ufw status numbered
```

**Tests (depuis vm1)** :

```bash
curl -I http://10.10.10.12:8080     # -> OK
curl -I http://10.10.10.12:80 || echo "Bloqué comme attendu"
```

> Comme on cible **enp0s8**, l’accès Internet via l’interface NAT reste OK.

---

## Scénario C — **Port précis** sur **vm3**

Objectifs :

* vm3 écoute **9090/TCP**.
* Autoriser **vm2 → vm3:9090**.
* Bloquer (log) **vm1 → vm3:9090**.

Sur **vm3** :

```bash
sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow in on enp0s8 from 10.10.10.12 to any port 9090 proto tcp
sudo ufw deny  log in on enp0s8 from 10.10.10.11 to any port 9090 proto tcp

sudo ufw enable
sudo ufw status numbered
```

**Tests :**

* Depuis **vm2** : `nc -zv -w 3 10.10.10.13 9090` → **OK**
* Depuis **vm1** : `nc -zv -w 3 10.10.10.13 9090` → **timeout (DROP)** et log dans `/var/log/ufw.log`

---

### (Option) Reset UFW si tu veux passer à iptables ensuite

```bash
sudo ufw --force reset
sudo ufw disable
```

---

# 🟠 Partie B — TP avec **iptables** (équivalent bas niveau)

⚠️ Ne lance **iptables** QUE si UFW est **désactivé/réinitialisé**.

> Rappel utile et sûr :

```bash
# Laisser passer les connexions déjà établies/relatives (bonne pratique)
sudo iptables -C INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT \
  || sudo iptables -I INPUT 1 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

## Scénario A — **Entrant** sur **vm2**

```bash
# SSH : autoriser vm1, bloquer vm3
sudo iptables -I INPUT 1 -i enp0s8 -p tcp --dport 22 -s 10.10.10.11 -j ACCEPT
sudo iptables -I INPUT 1 -i enp0s8 -p tcp --dport 22 -s 10.10.10.13 -j DROP

# HTTP 8080 : autoriser tout le lab
sudo iptables -A INPUT -i enp0s8 -p tcp --dport 8080 -j ACCEPT

# ICMP : autoriser vm1, bloquer vm3
sudo iptables -A INPUT -i enp0s8 -p icmp --icmp-type echo-request -s 10.10.10.11 -j ACCEPT
sudo iptables -A INPUT -i enp0s8 -p icmp --icmp-type echo-request -s 10.10.10.13 -j DROP
```

**Tests** : identiques à la partie UFW (depuis vm1 et vm3).

---

## Scénario B — **Sortant** sur **vm1**

```bash
# Autoriser d'abord vm2:8080
sudo iptables -I OUTPUT 1 -o enp0s8 -p tcp -d 10.10.10.12 --dport 8080 -j ACCEPT

# Bloquer HTTP (80/TCP) en sortie sur le lab
sudo iptables -A OUTPUT -o enp0s8 -p tcp --dport 80 -j DROP
```

**Tests (vm1)** :

```bash
curl -I http://10.10.10.12:8080      # -> OK
curl -I http://10.10.10.12:80 || echo "Bloqué comme attendu"
```

---

## Scénario C — **Port précis** sur **vm3**

```bash
# Autoriser vm2 -> 9090, bloquer vm1 -> 9090
sudo iptables -I INPUT 1 -i enp0s8 -p tcp --dport 9090 -s 10.10.10.12 -j ACCEPT
sudo iptables -A INPUT    -i enp0s8 -p tcp --dport 9090 -s 10.10.10.11 -j DROP

# (Option stricte) tout le reste vers 9090 est bloqué
# sudo iptables -A INPUT -i enp0s8 -p tcp --dport 9090 -j DROP
```

**Tests** :

* vm2 → vm3:9090 : **OK**
* vm1 → vm3:9090 : **timeout / refus** selon la règle

---

## Vérifications & logs (iptables)

```bash
# Compteurs/numéros des règles
sudo iptables -L INPUT  -n -v --line-numbers | egrep '22|8080|9090|icmp'
sudo iptables -L OUTPUT -n -v --line-numbers | egrep '80|8080'

# Observation trafic pendant un test
sudo tcpdump -ni enp0s8 'tcp port 8080 or tcp port 9090' -vv -c 5
```

*(Si tu veux des logs détaillés, ajoute une règle `-j LOG` juste avant le `DROP`, ex :)*

```bash
# Exemple sur vm3 : log puis drop vm1->9090
sudo iptables -I INPUT 1 -i enp0s8 -p tcp --dport 9090 -s 10.10.10.11 \
  -j LOG --log-prefix "IPT DROP 9090 vm1 " --log-level 4
sudo iptables -I INPUT 2 -i enp0s8 -p tcp --dport 9090 -s 10.10.10.11 -j DROP

# Voir les logs:
sudo journalctl -k -f
```

### (Option) Persistance des règles iptables

```bash
sudo apt -y install iptables-persistent
sudo netfilter-persistent save
```

### (Option) Reset rapide iptables (à utiliser en console locale)

```bash
# Remet les politiques par défaut à ACCEPT et vide les règles
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -F
sudo iptables -X
```

---

## Tableau d’attendus (résumé)

| Scénario         | Source → Cible     | Test                              | Attendu    |
| ---------------- | ------------------ | --------------------------------- | ---------- |
| A (vm2 IN) SSH   | vm1 → vm2:22       | `ssh vagrant@10.10.10.12`         | **OK**     |
| A (vm2 IN) SSH   | vm3 → vm2:22       | `ssh vagrant@10.10.10.12`         | **Bloqué** |
| A (vm2 IN) HTTP  | vm1/vm3 → vm2:8080 | `curl -I http://10.10.10.12:8080` | **OK**     |
| A (vm2 IN) ping  | vm1 → vm2          | `ping -c2 10.10.10.12`            | **OK**     |
| A (vm2 IN) ping  | vm3 → vm2          | `ping -c2 10.10.10.12`            | **Bloqué** |
| B (vm1 OUT) HTTP | vm1 → vm2:8080     | `curl -I http://10.10.10.12:8080` | **OK**     |
| B (vm1 OUT) HTTP | vm1 → vm2:80       | `curl -I http://10.10.10.12:80`   | **Bloqué** |
| C (vm3 IN) TCP   | vm2 → vm3:9090     | `nc -zv 10.10.10.13 9090`         | **OK**     |
| C (vm3 IN) TCP   | vm1 → vm3:9090     | `nc -zv 10.10.10.13 9090`         | **Bloqué** |

---

## Astuces & pièges fréquents

* **Ne filtre pas l’interface NAT** (souvent `enp0s3` en Vagrant/VirtualBox) → reste focalisé sur **`enp0s8`**.
* **Service non lancé ≠ pare-feu** : “Connection refused” ⇒ port fermé ; **DROP** ⇒ timeout.
* **UFW et iptables** : fais **l’un OU l’autre**. Si tu passes de UFW → iptables : `ufw --force reset && ufw disable`.
* **Voir ce qui écoute** : `sudo ss -ltnp` ; **voir les paquets** : `sudo tcpdump -ni enp0s8 ...`.

Si tu veux, je peux te fournir une **fiche “corrigé”** prête à imprimer (PDF) avec tous les blocs de commandes et la grille d’attendus.
