Très bon sujet 👍 tu veux ajouter la **protection SSH** à ton lab.
On va voir deux volets complémentaires :

1. **Fail2Ban** → détection des tentatives bruteforce dans les logs et bannissement automatique.
2. **Restriction d’accès par IP** → autoriser uniquement certaines adresses à se connecter en SSH.

---

# 🔒 1) Fail2Ban

👉 Fail2Ban surveille les logs (`/var/log/auth.log` en Ubuntu) et applique automatiquement des règles **iptables/ufw** pour bloquer une IP après plusieurs tentatives échouées.

## Installation (sur **vm2** par ex.)

```bash
sudo apt update
sudo apt install -y fail2ban
```

## Vérification

```bash
systemctl status fail2ban
```

## Configuration

Éditer (ou créer) `/etc/fail2ban/jail.local` :

```ini
[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 3
bantime  = 600
```

* `maxretry=3` → 3 tentatives échouées avant bannissement.
* `bantime=600` → bannissement 10 min.

## Test

1. Depuis **vm1**, tente une connexion SSH sur vm2 avec un mauvais mot de passe :

   ```bash
   ssh fakeuser@10.10.10.12
   ```

   (répéter 3 fois)
2. Vérifie sur vm2 :

   ```bash
   sudo fail2ban-client status sshd
   ```

   → l’IP de vm1 doit apparaître comme bannie.
3. Depuis vm1 → la connexion SSH est refusée (DROP).

---

# 🌐 2) Restriction SSH par IP

👉 Ici, on autorise seulement certaines IP à se connecter au service SSH.

## Méthode A – avec `sshd_config`

Dans `/etc/ssh/sshd_config` sur vm2 :

```conf
AllowUsers vagrant@10.10.10.11
```

👉 seul `vm1 (10.10.10.11)` pourra se connecter.

Puis :

```bash
sudo systemctl restart ssh
```

## Méthode B – avec `iptables`

Sur vm2, autoriser SSH seulement depuis vm1 :

```bash
sudo iptables -A INPUT -p tcp -s 10.10.10.11 --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j DROP
```

👉 Résultat :

* Depuis vm1 → connexion SSH OK.
* Depuis vm3 → connexion refusée.

## Méthode C – avec `ufw`

Sur vm2 :

```bash
sudo ufw allow from 10.10.10.11 to any port 22
sudo ufw deny 22/tcp
```

---

# 🧪 3) Exercices pour ton lab

### Exercice 1 : Fail2Ban

* Configurer Fail2Ban sur vm2.
* Depuis vm1 → tenter 3 connexions SSH ratées.
* Vérifier que vm1 est banni.

### Exercice 2 : Restriction IP

* Sur vm2, autoriser SSH uniquement depuis vm1.
* Vérifier :

  * vm1 → `ssh vagrant@10.10.10.12` → OK.
  * vm3 → tentative échoue.

### Exercice 3 (bonus)

* Combiner Fail2Ban + restriction IP.
* Résultat : seul vm1 peut se connecter, et en cas d’attaques, Fail2Ban bannit automatiquement l’IP.

---

# ✅ Résumé pédagogique

* **Fail2Ban** → défense automatique contre brute force (logs + bannissement IP).
* **Restrictions IP** → contrôle strict de qui peut se connecter en SSH.
* Ensemble, ces mécanismes font partie des **bonnes pratiques de sécurisation SSH**.

---

Veux-tu que je te prépare un **TP complet “Protection SSH”** (énoncé + corrigé) où les apprenants installent Fail2Ban, provoquent un bannissement, puis configurent des restrictions IP pour n’autoriser que vm1 à se connecter sur vm2 ?
