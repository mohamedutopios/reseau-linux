Parfait 👍 je vais préciser **clairement** sur quelle VM tu fais chaque installation/exercice.
Avec ton lab :

* **vm1 = 10.10.10.11**
* **vm2 = 10.10.10.12**
* **vm3 = 10.10.10.13**

---

# 🔧 TP Réseau avec `netstat`, `ss`, `lsof`

## 1) Vérifier le serveur **SSH**

👉 **Sur toutes les VMs (vm1, vm2, vm3)** :

```bash
ss -tulpn | grep ssh
```

ou

```bash
systemctl status ssh
```

Attendu : `sshd` écoute sur le port 22.
→ C’est **déjà présent par défaut** (pas besoin d’installation).

---

## 2) Installer un serveur **Web Apache**

👉 **Sur vm3 (10.10.10.13)** :

```bash
sudo apt-get update
sudo apt-get install -y apache2
```

Vérification sur vm3 :

```bash
ss -tulpn | grep :80
sudo lsof -i :80
```

Puis test depuis **vm1** ou **vm2** :

```bash
curl http://10.10.10.13
```

Attendu : page HTML par défaut d’Apache.

---

## 3) Lancer un service **UDP** sur le port 5000

👉 **Sur vm2 (10.10.10.12)** :

```bash
nc -lu 5000
```

👉 **Depuis vm1 (10.10.10.11)** :

```bash
echo "hello depuis vm1" | nc -u 10.10.10.12 5000
```

👉 Vérification sur **vm2** (pendant que `nc` tourne) :

```bash
ss -un | grep 5000
sudo lsof -iUDP:5000
```

Attendu : socket UDP ouverte sur port 5000, et réception du message.

---

## 🔎 Objectifs pédagogiques

* **SSH (vm1/vm2/vm3)** → service natif déjà présent → observer connexions établies.
* **Apache (vm3)** → service à installer → observer écoute sur port 80 et connexions HTTP.
* **UDP (vm2)** → service temporaire via netcat → observer socket UDP + communication depuis vm1.

---

👉 Ainsi, chaque VM a un rôle précis :

* **vm1** : client de test (ping, curl, netcat).
* **vm2** : serveur UDP 5000.
* **vm3** : serveur Web Apache.
* **SSH** : présent sur toutes.

Veux-tu que je t’écrive maintenant un **corrigé complet** avec toutes les commandes `netstat`, `ss`, `lsof` à exécuter et ce qu’on doit voir en sortie (comme si c’était une fiche de TP étudiant) ?
