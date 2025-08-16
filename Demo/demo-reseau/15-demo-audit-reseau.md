Parfait 👍 tu veux terminer avec la partie **Audit réseau** dans ton lab, en utilisant **nmap** pour la découverte et l’analyse des services.
C’est une excellente suite après le pare-feu et la sécurité SSH.

---

# 🌍 1) Qu’est-ce que `nmap` ?

* **nmap** (*Network Mapper*) est un outil d’audit réseau qui permet :

  * Découverte d’hôtes (ping scan, ARP scan).
  * Détection de ports ouverts/fermés/filtrés.
  * Identification de services et versions.
  * Détection d’OS et fingerprinting.
  * Scripts NSE (Nmap Scripting Engine) pour tester des vulnérabilités.

---

# ⚙️ 2) Installation

👉 Sur **vm1** (qui jouera le rôle d’“attaquant/auditeur”) :

```bash
sudo apt update
sudo apt install -y nmap
```

---

# 🔎 3) Découverte d’hôtes (ping scan)

👉 Depuis vm1, scanner ton réseau privé `10.10.10.0/24` :

```bash
nmap -sn 10.10.10.0/24
```

✅ Résultat attendu :

* vm1 (10.10.10.11), vm2 (10.10.10.12), vm3 (10.10.10.13).
* Peut aussi afficher l’adresse MAC (utile pour distinguer les hôtes).

---

# 🔑 4) Scan de ports

## Ports standards

```bash
nmap 10.10.10.12
```

👉 Par défaut → ports les plus connus (1000 ports TCP).
Exemple attendu sur vm2 : port 22 (SSH) ouvert.

## Scan complet

```bash
nmap -p- 10.10.10.13
```

👉 Scanne **tous les 65 535 ports** → utile pour voir s’il y a d’autres services.
Exemple attendu : 22 (SSH) et 80 (Apache si installé).

---

# 🔬 5) Détection de services et versions

```bash
nmap -sV 10.10.10.13
```

👉 Essaye d’identifier les services (exemple : `OpenSSH 8.x`, `Apache httpd 2.4.x`).

---

# 🖥️ 6) Détection d’OS

```bash
sudo nmap -O 10.10.10.12
```

👉 Essaye d’identifier la version de l’OS (Linux Ubuntu).
⚠️ Résultats parfois approximatifs si pare-feu actif.

---

# 📜 7) Scripts NSE (Nmap Scripting Engine)

👉 Exemple : vérifier les vulnérabilités SSH sur vm2 :

```bash
nmap --script ssh* -p 22 10.10.10.12
```

👉 Exemple : détection http sur vm3 (Apache) :

```bash
nmap --script http-enum -p 80 10.10.10.13
```

---

# 🧪 8) Exercices pratiques dans ton lab

### Exercice 1 : Découverte

* Depuis vm1 → `nmap -sn 10.10.10.0/24`.
* Identifier les IP et MAC de vm2/vm3.

### Exercice 2 : Ports ouverts

* Scanner vm2 (`nmap 10.10.10.12`) → trouver SSH.
* Scanner vm3 (`nmap 10.10.10.13`) → trouver SSH + HTTP.

### Exercice 3 : Version des services

* Sur vm2 → identifier version d’OpenSSH.
* Sur vm3 → identifier version d’Apache.

### Exercice 4 : Effet d’un pare-feu

* Active ufw sur vm3 → autorise uniquement port 80.
* Depuis vm1, refaire `nmap` sur vm3.
* Vérifier que le port 22 apparaît comme **“filtered”** ou **fermé**.

### Exercice 5 (bonus) : Scripts NSE

* Lancer `nmap --script ssh2-enum-algos -p 22 10.10.10.12` pour voir les algorithmes SSH acceptés.
* Lancer `nmap --script http-title -p 80 10.10.10.13` pour lire le titre de la page web.

---

# ✅ Résumé pédagogique

* **nmap** = découverte et analyse de services réseau.
* Permet d’illustrer la différence entre **ports ouverts/fermés/filtrés**.
* Montre l’impact des règles **iptables/ufw** dans ton lab.
* Sert à comprendre l’**attaque (audit)** et la **défense (pare-feu, durcissement)**.

---

Veux-tu que je prépare un **TP complet “Audit réseau avec nmap”** (énoncé + corrigé) où les apprenants :

1. découvrent le réseau,
2. analysent les services,
3. comparent avant/après activation d’un pare-feu,
4. utilisent des scripts NSE ?
