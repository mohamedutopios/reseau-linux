Super idée 👍 tu veux déplacer ton **lab Hydra+Fail2ban** de `vm1` vers une relation **vm2 ⇄ vm3** :

* **vm3 (10.10.10.13)** → **serveur SSH** protégé par Fail2ban
* **vm2 (10.10.10.12)** → attaquant avec Hydra
* **vm1 (10.10.10.11)** → observateur/test

---

# 🛠 TP : Hydra vs Fail2ban (vm2 → vm3)

---

## 🔹 1) Installer et activer SSH sur vm3 (serveur)

Sur **vm3** :

```bash
sudo apt update && sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
```

Test depuis vm2 que la connexion marche :

```bash
ssh vagrant@10.10.10.13
```

---

## 🔹 2) Installer Fail2ban sur vm3

Toujours sur **vm3** :

```bash
sudo apt install fail2ban -y
```

Configurer `/etc/fail2ban/jail.local` :

```ini
[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 3       # 3 tentatives ratées max
bantime  = 300     # ban 5 minutes
findtime = 300     # fenêtre de 5 minutes
```

Active :

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd
```

---

## 🔹 3) Installer Hydra sur vm2 (attaquant)

Sur **vm2** :

```bash
sudo apt update && sudo apt install hydra -y
```

Préparer une wordlist simple `/tmp/passlist.txt` :

```bash
echo "123456"   > /tmp/passlist.txt
echo "password" >> /tmp/passlist.txt
echo "toto"     >> /tmp/passlist.txt
```

---

## 🔹 4) Lancer l’attaque brute force (vm2 → vm3)

Toujours sur **vm2** :

```bash
hydra -l vagrant -P /tmp/passlist.txt ssh://10.10.10.13
```

👉 Après **3 échecs**, Fail2ban sur **vm3** bannira l’IP de vm2.

---

## 🔹 5) Vérifier le bannissement (sur vm3)

```bash
sudo fail2ban-client status sshd
```

Tu devrais voir :

```
Banned IP list: 10.10.10.12
```

---

## 🔹 6) Tester le blocage

Sur **vm2** :

```bash
ssh vagrant@10.10.10.13
```

👉 Refus immédiat (plus de demande de mot de passe).

Sur **vm1** (observateur) :

```bash
ssh vagrant@10.10.10.13
```

👉 Connexion OK (Fail2ban n’a bloqué que vm2).

---

## 🔹 7) Débannir vm2 (si besoin)

Sur **vm3** :

```bash
sudo fail2ban-client set sshd unbanip 10.10.10.12
```

---

# ✅ Résultat attendu

* vm2 attaque vm3 en SSH avec Hydra
* Fail2ban bannit automatiquement vm2 après 3 tentatives ratées
* vm2 n’a plus accès à vm3
* vm1 reste autorisé (non banni)

---

Veux-tu que je te prépare aussi la **lecture et explication des logs `/var/log/auth.log` sur vm3** pour voir les tentatives Hydra + l’action de Fail2ban ?
