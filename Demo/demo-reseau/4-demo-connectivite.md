Parfait 👍 tu veux compléter ton lab avec une **partie “Vérification de la connectivité”**.
Avec **ICMP**, **ports ouverts**, et **état des sockets**, voici un plan clair **par VM** avec les commandes typiques et ce qu’on doit observer :

---

# 🔧 Vérification de la connectivité

## 1) **ICMP (ping)**

👉 **Depuis vm1 (10.10.10.11)** :

* Vérifier connectivité LAN :

```bash
ping -c 3 10.10.10.12   # vm2
ping -c 3 10.10.10.13   # vm3
```

* Vérifier sortie Internet (via NAT) :

```bash
ping -c 3 1.1.1.1
```

* Test MTU :

```bash
ping -M do -s 1472 1.1.1.1
```

✅ Attendu : réponses ICMP → connectivité OK.
Si KO → vérifier `ip addr`, `ip link`, `ip route`.

---

## 2) **Vérifier les ports ouverts (TCP/UDP)**

### 🔑 SSH (déjà actif sur toutes les VMs)

👉 **Sur vm2 (10.10.10.12)** :

```bash
ss -tulpn | grep :22
sudo lsof -i :22
```

✅ Attendu : `sshd` en écoute sur `0.0.0.0:22`.

### 🌐 Web Apache (à installer sur vm3)

👉 **Sur vm3 (10.10.10.13)** :

```bash
sudo apt-get install -y apache2
ss -tulpn | grep :80
sudo lsof -i :80
```

👉 **Depuis vm1** :

```bash
curl http://10.10.10.13
```

✅ Attendu : Apache en écoute sur `:80` + page par défaut reçue.

### 📡 UDP 5000 (via netcat sur vm2)

👉 **Sur vm2** :

```bash
nc -lu 5000
```

👉 **Depuis vm1** :

```bash
echo "Hello UDP" | nc -u 10.10.10.12 5000
```

👉 **Toujours sur vm2** (dans un 2ᵉ terminal) :

```bash
ss -un | grep 5000
sudo lsof -iUDP:5000
```

✅ Attendu : socket UDP ouverte sur `:5000`, réception du message.

---

## 3) **État des sockets (connexions actives)**

👉 **Depuis vm1 (quand tu te connectes à vm3 en HTTP)** :

```bash
ss -tn
```

Exemple attendu :

```
ESTAB  0  0  10.10.10.11:45678  10.10.10.13:80
```

👉 **Depuis vm3 (serveur web)** :

```bash
ss -tn sport = :80
```

Exemple attendu :

```
ESTAB  0  0  10.10.10.13:80  10.10.10.11:45678
```

👉 Pour l’UDP (vm2) :

```bash
ss -un
```

Exemple attendu :

```
UNCONN 0 0 0.0.0.0:5000  *:*
```

---

# ✅ Résumé pédagogique

* **ICMP (ping)** → tester la connectivité IP (LAN + Internet).
* **Ports ouverts** → vérifier avec `ss -tulpn` et `lsof -i`.
* **Sockets** → suivre les connexions établies (`ESTAB` TCP, `UNCONN` UDP).

👉 Dans ton lab :

* **vm1** = poste client/test.
* **vm2** = serveur SSH + UDP (5000).
* **vm3** = serveur SSH + Apache (80).

---

Veux-tu que je t’écrive maintenant un **scénario de TP complet “diagnostic réseau”** (énoncé + corrigé), où l’apprenant doit prouver pas à pas la connectivité ICMP, l’écoute des ports, et l’état des sockets ?
