Parfait 👍 tu veux ajouter un **Serveur DNS** à ton lab (vm1/vm2/vm3).
Deux approches classiques sous Linux :

* **bind9** → serveur DNS complet, robuste, utilisé en prod (plus complexe).
* **dnsmasq** → petit serveur DNS + DHCP, léger, parfait pour un lab.

---

# 🌐 1) Installation

👉 Sur **vm2** (qui pourra être ton DNS central) :

### Option A – `bind9`

```bash
sudo apt update
sudo apt install -y bind9 bind9-utils
```

### Option B – `dnsmasq`

```bash
sudo apt update
sudo apt install -y dnsmasq
```

---

# ⚙️ 2) Configuration avec **bind9**

## Étape A – fichier de configuration principal

Éditer `/etc/bind/named.conf.options` :

```conf
options {
    directory "/var/cache/bind";

    # DNS publics utilisés si la zone n’est pas connue
    forwarders {
        1.1.1.1;
        8.8.8.8;
    };

    dnssec-validation auto;
    listen-on { any; };
    allow-query { any; };
};
```

## Étape B – définir une zone locale `lab.local`

Éditer `/etc/bind/named.conf.local` :

```conf
zone "lab.local" {
    type master;
    file "/etc/bind/db.lab.local";
};
```

## Étape C – créer le fichier de zone

Créer `/etc/bind/db.lab.local` :

```conf
$TTL    604800
@       IN      SOA     vm2.lab.local. admin.lab.local. (
                        1         ; Serial
                        604800    ; Refresh
                        86400     ; Retry
                        2419200   ; Expire
                        604800 )  ; Negative Cache TTL
;
@       IN      NS      vm2.lab.local.

vm1     IN      A       10.10.10.11
vm2     IN      A       10.10.10.12
vm3     IN      A       10.10.10.13
```

## Étape D – redémarrer et tester

```bash
sudo systemctl restart bind9
dig @10.10.10.12 vm1.lab.local
dig @10.10.10.12 www.google.com
```

---

# ⚙️ 3) Configuration avec **dnsmasq** (plus simple)

## Étape A – sauvegarde et édition

Éditer `/etc/dnsmasq.conf` :

```conf
# Interface sur laquelle dnsmasq écoute
interface=enp0s8

# Domaine local
domain=lab.local

# Entrées statiques
address=/vm1.lab.local/10.10.10.11
address=/vm2.lab.local/10.10.10.12
address=/vm3.lab.local/10.10.10.13

# Redirection vers DNS publics
server=1.1.1.1
server=8.8.8.8
```

## Étape B – redémarrage

```bash
sudo systemctl restart dnsmasq
```

## Étape C – test

Depuis **vm1** (après avoir pointé vers vm2 comme DNS) :

```bash
dig @10.10.10.12 vm3.lab.local
ping vm2.lab.local
```

---

# 🛠️ 4) Clients (vm1 et vm3)

👉 Modifier `/etc/resolv.conf` pour utiliser **vm2 comme DNS** :

```conf
nameserver 10.10.10.12
```

👉 Tester :

```bash
dig vm2.lab.local
ping vm1.lab.local
```

---

# 🧪 5) Exercices pratiques

1. **Résolution locale**

   * Depuis vm1, résoudre `vm3.lab.local` via vm2 (serveur DNS).

2. **Forwarding externe**

   * Depuis vm1, résoudre `www.google.com` via vm2.

3. **Ajout d’un nouvel enregistrement**

   * Ajouter `web.lab.local` pointant sur `10.10.10.13`.
   * Tester avec `dig` depuis vm1.

---

# ✅ Résumé pédagogique

* **bind9** = serveur DNS complet, zones avancées, proche du monde réel.
* **dnsmasq** = rapide à configurer, parfait pour un lab léger.
* Dans ton lab :

  * **vm2** = serveur DNS.
  * **vm1/vm3** = clients, utilisent vm2 comme résolveur.

---

👉 Veux-tu que je te prépare un **TP complet DNS** (énoncé + corrigé) basé sur `dnsmasq` (rapide pour ton lab), et un autre basé sur `bind9` (plus réaliste entreprise) ?
