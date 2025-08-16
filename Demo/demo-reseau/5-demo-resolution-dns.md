Parfait 👍 tu veux ajouter la **partie Résolution de noms** dans ton lab.
On va voir **le fonctionnement DNS**, puis comment tester/modifier la résolution avec `/etc/hosts` et `/etc/resolv.conf` sur tes VMs Ubuntu (vm1/vm2/vm3).

---

# 🌍 1) Fonctionnement du DNS

* Quand tu tapes `ping www.example.com`, le système doit traduire le **nom de domaine** → **adresse IP**.
* Par défaut :

  1. Le résolveur local consulte `/etc/hosts` (fichier statique).
  2. Sinon, il utilise les serveurs DNS listés dans `/etc/resolv.conf`.
  3. Si aucun ne répond, résolution échoue.

👉 Sur Ubuntu avec Vagrant + VirtualBox, le **DNS par défaut** est souvent celui fourni par VirtualBox (NAT) → redirigé vers les DNS de ta machine hôte.

---

# 📝 2) Modification de `/etc/hosts`

👉 **But : créer une résolution locale pour tes VMs (vm1/vm2/vm3)**.

### Étape :

Édite `/etc/hosts` sur **chaque VM** (root nécessaire) :

```bash
sudo nano /etc/hosts
```

Ajoute :

```
10.10.10.11   vm1.lab   vm1
10.10.10.12   vm2.lab   vm2
10.10.10.13   vm3.lab   vm3
```

### Test (depuis vm1) :

```bash
ping -c 2 vm2.lab
nslookup vm3.lab
```

✅ Résultat attendu : résolution directe, même sans DNS externe.

---

# ⚙️ 3) Modification de `/etc/resolv.conf`

👉 Ce fichier définit **quel(s) serveur(s) DNS** utiliser.

### Vérifier contenu actuel :

```bash
cat /etc/resolv.conf
```

Tu verras quelque chose comme :

```
nameserver 10.0.2.3
```

(`10.0.2.3` est fourni par VirtualBox NAT → renvoie vers les DNS de ton hôte).

### Changer temporairement de DNS :

👉 Exemple, utiliser Cloudflare ou Google :

```bash
sudo nano /etc/resolv.conf
```

Mettre :

```
nameserver 1.1.1.1
nameserver 8.8.8.8
```

### Test :

```bash
dig www.openai.com
nslookup www.google.com
```

✅ Résultat attendu : adresse(s) IP publique(s).

⚠️ Attention : Sur Ubuntu 22.04, `/etc/resolv.conf` est souvent géré par **systemd-resolved** → tes modifs peuvent être écrasées au reboot.
👉 Pour une modif **persistante** :

```bash
sudo nano /etc/systemd/resolved.conf
```

Ajoute/modifie :

```
[Resolve]
DNS=1.1.1.1 8.8.8.8
FallbackDNS=9.9.9.9
```

Puis :

```bash
sudo systemctl restart systemd-resolved
```

---

# 🧪 4) Exercices dans ton lab

1. **Hosts statiques**

   * Ajouter `vm1.lab`, `vm2.lab`, `vm3.lab` dans `/etc/hosts`.
   * Vérifier avec `ping` et `nslookup`.

2. **DNS public**

   * Modifier `/etc/resolv.conf` → `nameserver 1.1.1.1`.
   * Tester `dig www.wikipedia.org`.

3. **DNS cassé**

   * Mets un faux serveur DNS (`nameserver 10.10.10.254`).
   * Lance `ping www.google.com` → échec attendu.
   * Vérifie que `ping vm2.lab` fonctionne toujours grâce à `/etc/hosts`.

---

# ✅ Résumé pédagogique

* **DNS = traduction noms ↔ IP**, via `/etc/hosts` (local) puis `/etc/resolv.conf` (DNS externe).
* **/etc/hosts** = utile pour ton LAN lab (vm1/vm2/vm3).
* **/etc/resolv.conf** = utile pour forcer ou tester différents serveurs DNS.

---

Veux-tu que je t’écrive un **TP complet “Résolution de noms”** (énoncé + corrigé) qui combine :

1. résolution interne avec `/etc/hosts`,
2. résolution Internet avec DNS publics,
3. simulation d’une panne DNS ?
