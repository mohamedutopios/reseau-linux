Parfait 👍 tu veux mettre en place une **connexion SSH sans mot de passe** de **vm2 → vm3** avec une clé SSH.
Voici le pas-à-pas clair 👇

---

# 🔑 1) Générer une clé SSH sur **vm3** (côté serveur → futur accès)

Sur **vm3** (10.10.10.13), en tant que `vagrant` :

```bash
ssh-keygen -t ed25519 -C "vagrant@vm3"
```

* Quand il demande *"Enter file in which to save the key"*, laisse par défaut (`~/.ssh/id_ed25519`) → juste **Entrée**.
* Quand il demande une *passphrase*, tu peux laisser vide (pas de mot de passe).

Cela crée :

* **\~/.ssh/id\_ed25519** → clé privée (garde-la secrète 🔒)
* **\~/.ssh/id\_ed25519.pub** → clé publique (celle qu’on partage)

---

# 🔑 2) Copier la clé publique vers **vm2**

Toujours sur **vm3** :

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub vagrant@10.10.10.12
```

⚠️ La première fois il demandera le mot de passe de `vagrant` sur **vm2** (par défaut `vagrant`, sauf si tu l’as changé).

---

# 🔑 3) Vérifier depuis **vm3 → vm2**

Depuis **vm3** :

```bash
ssh vagrant@10.10.10.12
```

👉 Si tout est bien configuré, tu rentres **directement sans mot de passe** ✅.

---

# 📌 Variante manuelle (si pas de `ssh-copy-id`)

Si `ssh-copy-id` n’est pas installé, fais à la main :

Sur **vm3** :

```bash
cat ~/.ssh/id_ed25519.pub
```

Copie la ligne complète.

Sur **vm2** :

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys   # colle la clé ici
chmod 600 ~/.ssh/authorized_keys
```

---

# 🔄 Résumé

* **Clé générée** sur vm3
* **Clé publique copiée** sur vm2 (`authorized_keys`)
* Résultat : **connexion ssh vm3 → vm2 sans mot de passe**

---

👉 Veux-tu que je prépare aussi le scénario inverse (**vm2 → vm3**) pour ton lab Hydra/Fail2ban, afin de tester attaque par mot de passe vs accès par clé ?
