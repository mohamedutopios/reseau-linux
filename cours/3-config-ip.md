Parfait 👌 tu veux un focus pédagogique sur la **configuration réseau IP statique et DHCP sous Linux**, avec les **méthodes classiques**.
Je vais détailler les deux grands cas : **ancienne méthode (`/etc/network/interfaces`)** et **méthode moderne (NetworkManager / Netplan selon distro)**.

---

# ⚙️ Configuration IP statique et DHCP sous Linux

---

## 🔹 1. Avec `/etc/network/interfaces` (Debian/Ubuntu "classiques")

📌 Utilisé surtout sur les versions **anciennes** de Debian/Ubuntu.

* **Fichier :** `/etc/network/interfaces`

### Exemple : IP statique

```ini
# Interface loopback
auto lo
iface lo inet loopback

# Interface Ethernet en IP statique
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 1.1.1.1
```

### Exemple : DHCP

```ini
auto eth0
iface eth0 inet dhcp
```

### Commandes pour activer/désactiver

```bash
sudo ifdown eth0    # désactiver
sudo ifup eth0      # activer
```

---

## 🔹 2. Avec **Netplan** (Ubuntu 18.04+)

📌 Méthode moderne sur Ubuntu.

* **Fichiers :** `/etc/netplan/*.yaml`

### Exemple : IP statique

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

### Exemple : DHCP

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: yes
```

### Application des changements

```bash
sudo netplan apply
```

---

## 🔹 3. Avec **NetworkManager** (`nmcli`)

📌 Méthode pratique sur Fedora, RHEL, CentOS, Ubuntu Desktop.

### Lister les connexions

```bash
nmcli con show
```

### Créer une configuration IP statique

```bash
nmcli con add type ethernet ifname eth0 con-name eth0-static ipv4.addresses 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.dns "8.8.8.8" ipv4.method manual
```

### Passer une interface en DHCP

```bash
nmcli con mod eth0 ipv4.method auto
```

### Activer/désactiver l’interface

```bash
nmcli con down eth0
nmcli con up eth0
```

---

## 🔹 4. Autres outils utiles

* **`ip link set eth0 up/down`** → activer/désactiver une interface manuellement
* **`systemctl restart NetworkManager`** → redémarrer le service
* **`resolvectl status`** (systemd-resolved) → état DNS

---

# ✅ Résumé pratique (pour TP)

* **DHCP (interfaces) :**

  ```ini
  auto eth0
  iface eth0 inet dhcp
  ```

* **Statique (Netplan) :**

  ```yaml
  addresses: [192.168.1.100/24]
  gateway4: 192.168.1.1
  dns-nameservers: [8.8.8.8, 1.1.1.1]
  ```

* **Activation/désactivation :**

  ```bash
  sudo ifdown eth0 && sudo ifup eth0
  sudo ip link set eth0 down
  sudo ip link set eth0 up
  sudo nmcli con down eth0 && sudo nmcli con up eth0
  ```

---

👉 Veux-tu que je t’écrive un **TP complet** où l’apprenant :

1. Configure une interface en **DHCP** puis teste la connectivité.
2. Passe la même interface en **IP statique** (édition fichiers).
3. Utilise `ping`, `ip addr`, et `ip route` pour vérifier ?
