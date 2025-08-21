Parfait 👍 tu veux un **TP clair en 3 scénarios distincts**, bien complets, avec toutes les étapes bien reliées, sans “trous”.

Contexte :

* **vm1 = 10.10.10.11**
* **vm2 = 10.10.10.12**
* **vm3 = 10.10.10.13**
* Utilisateur par défaut : **vagrant**
* Hôte Windows avec Vagrant + VirtualBox

---

# 🟢 Scénario 1 — Connexion depuis l’hôte avec **clé SSH uniquement** (vers vm1)

👉 Objectif : interdire le mot de passe et n’autoriser que l’authentification par clé SSH depuis l’hôte.

### Étapes

**Sur l’hôte Windows :**

```powershell
# 1. Générer une nouvelle paire de clés
ssh-keygen -t ed25519 -f C:\Users\mohamed\Documents\Formation\reseau-linux\installation\vagrant\ma_cle_vm1 -C "mohamed@host"
```

➡️ Cela crée deux fichiers :

* `ma_cle_vm1` (clé privée, reste sur ton hôte Windows)
* `ma_cle_vm1.pub` (clé publique, à copier dans la VM)

**Sur vm1 (via vagrant ssh) :**

```bash
vagrant ssh vm1
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cat /vagrant/ma_cle_vm1.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

**Configurer SSH pour n’accepter que la clé :**

```bash
sudo nano /etc/ssh/sshd_config
```

Modifie/ajoute :

```
PasswordAuthentication no
PubkeyAuthentication yes
```

Puis redémarre SSH :

```bash
sudo systemctl restart ssh
```

**Test depuis l’hôte Windows :**

```powershell
ssh -i C:\Users\mohamed\Documents\Formation\reseau-linux\installation\vagrant\ma_cle_vm1 -p 2222 vagrant@127.0.0.1
```

✅ Résultat : connexion réussie avec clé, mot de passe refusé.

---

# 🟡 Scénario 2 — Connexion avec **mot de passe uniquement** (vers vm2)

👉 Objectif : désactiver la clé SSH et n’autoriser que le mot de passe.

### Étapes

**Sur vm2 (via `vagrant ssh vm2`) :**

```bash
# Donner un mot de passe à l’utilisateur vagrant
sudo passwd vagrant
# (choisis par ex. vagrant)
```

**Configurer SSH pour n’accepter que le mot de passe :**

```bash
sudo nano /etc/ssh/sshd_config
```

Modifie/ajoute :

```
PasswordAuthentication yes
PubkeyAuthentication no
```

Puis redémarre SSH :

```bash
sudo systemctl restart ssh
```

**Test depuis l’hôte Windows :**

```powershell
ssh -p 2200 vagrant@127.0.0.1
```

➡️ Ça doit demander le **mot de passe**, et toute clé SSH sera refusée.

✅ Résultat : connexion avec mot de passe uniquement.

---

# 🔴 Scénario 3 — Connexion root **autorisée** (vers vm3)

👉 Objectif : permettre de se connecter directement en root (ce qui est normalement interdit par Vagrant).

### Étapes

**Sur vm3 (via `vagrant ssh vm3`) :**

```bash
# Définir un mot de passe root
sudo passwd root
# (mets un mot de passe fort, ex : Root123!)
```

**Configurer SSH pour autoriser root :**

```bash
sudo nano /etc/ssh/sshd_config
```

Modifie/ajoute :

```
PermitRootLogin yes
PasswordAuthentication yes
```

Puis redémarre SSH :

```bash
sudo systemctl restart ssh
```

**Test depuis l’hôte Windows :**

```powershell
ssh -p 2201 root@127.0.0.1
```

➡️ Entre le mot de passe root que tu viens de définir.

✅ Résultat : tu peux te connecter directement en root.

---

# 🎯 Résumé

* **vm1** → uniquement **clé SSH** (pas de mot de passe)
* **vm2** → uniquement **mot de passe** (clé refusée)
* **vm3** → **root autorisé** en SSH avec mot de passe

---

👉 Veux-tu que je prépare aussi le **TP inverse** :

* vm1 = mot de passe uniquement
* vm2 = clé SSH uniquement
* vm3 = root interdit (défaut, mais avec test explicite)
  … pour que tes apprenants comparent les 2 configs ?
