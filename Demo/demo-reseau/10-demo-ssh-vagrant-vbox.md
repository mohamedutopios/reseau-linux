Yes 👌 tu es exactement dans le cas où tu lances **ssh manuellement** au lieu de `vagrant ssh`.

👉 Voici ce qui se passe :

1. **Authentification Vagrant**

   * Les VMs Vagrant sont configurées pour n’accepter que **l’authentification par clé SSH**.
   * L’utilisateur `vagrant` n’a **pas de mot de passe** utilisable en SSH.
   * La clé privée se trouve dans `~/.vagrant.d/insecure_private_key` (ou une clé générée si tu as reprovisionné).
   * Quand tu fais `vagrant ssh`, Vagrant ajoute automatiquement la bonne clé et le bon port.

2. **Ton erreur**

   * Tu t’es connecté sur le bon port (`-p 2222`)
   * Mais `ssh` ne sait pas quelle clé utiliser → donc tu tombes sur **`Permission denied (publickey)`**.

---

## ✅ Solutions

### 🔹 1) Utiliser la commande Vagrant (le plus simple)

```powershell
vagrant ssh vm1
vagrant ssh vm2
vagrant ssh vm3
```

> Vagrant détecte automatiquement le port NAT et charge la clé privée.

---

### 🔹 2) Utiliser ssh avec la clé fournie par Vagrant

* Récupère la clé privée :

```powershell
vagrant ssh-config vm1
```

Exemple de sortie :

```
Host vm1
  HostName 127.0.0.1
  User vagrant
  Port 2222
  IdentityFile C:/Users/mohamed/Documents/Formations/reseau-linux/installation/vagrant/.vagrant/machines/vm1/virtualbox/private_key
```

* Connecte-toi en SSH avec :

```powershell
ssh -i .vagrant/machines/vm1/virtualbox/private_key -p 2222 vagrant@127.0.0.1
```

---

### 🔹 3) (Option) Forcer un mot de passe pour l’utilisateur `vagrant`

👉 Pas recommandé pour Vagrant, mais utile pour tes tests entre VMs.
Sur chaque VM :

```bash
sudo passwd vagrant
```

puis mets un mot de passe (ex. `vagrant`).
Ensuite tu pourras faire depuis une autre VM :

```bash
ssh vagrant@10.10.10.12
```

et entrer le mot de passe.

---

## 🎯 Résumé

* Depuis **Windows (hôte)** : utilise `vagrant ssh` ou `ssh -i <private_key>` sur le port redirigé (2222/2200/2201).
* Depuis **une autre VM** : tu peux utiliser `ssh vagrant@10.10.10.x` **si tu ajoutes un mot de passe** ou copies la clé publique sur les autres VMs.

---

👉 Veux-tu que je te prépare un **TP clé en main** pour :

1. Activer un mot de passe SSH simple pour `vagrant` entre tes VMs
2. Et une version plus propre avec **copie de clé publique** (connexion sans mot de passe entre vm1/vm2/vm3) ?
