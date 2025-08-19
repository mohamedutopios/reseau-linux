Parfait — on garde ton objectif (3 VMs en 10.10.10.0/24) et on **garde le Vagrantfile tel quel**.
Ci-dessous tu as la **procédure complète**, **annotée** avec le **pourquoi** de chaque action, des **exemples**, des **contrôles**, et des **démos**.

---

# ⚙️ Vue d’ensemble

* **vm1** devient **serveur DHCP** (isc-dhcp-server) sur l’interface **du lab** (host-only, typiquement `enp0s8`).
* **vm2** et **vm3** passent l’interface du lab de **statique → DHCP** **à l’intérieur** des VMs (netplan).
* On **évite** `vagrant reload/destroy` pendant le TP (Vagrant remettrait l’IP statique d’origine).

---

# Étape 0 — Vérifs côté hôte (VirtualBox)

```bash
# (sur l'hôte)
VBoxManage list hostonlyifs
VBoxManage list dhcpservers
```

**Pourquoi ?**

* Confirmer que le réseau **host-only** de VirtualBox **n’a pas** déjà un DHCP (sinon il ferait concurrence à vm1).
* En pratique, `private_network` de Vagrant **n’active pas** ce DHCP, donc tu devrais ne **rien** voir pour ce segment.

---

# Étape 1 — vm1 en serveur DHCP

## 1.1 Installer et cibler la bonne interface

```bash
vagrant ssh vm1
sudo apt-get update
sudo apt-get install -y isc-dhcp-server
ip -o link show | awk -F': ' '{print $2}'    # repérer le nom de l'interface lab (souvent enp0s8)
ip -4 addr show enp0s8                        # vérifier qu'elle a bien 10.10.10.11/24
```

**Pourquoi ?**

* `isc-dhcp-server` doit **écouter** sur **l’interface du lab** (pas sur la NAT).
* Si tu ne le lies pas à `enp0s8`, soit il refuse de démarrer, soit il pourrait annoncer du DHCP sur la mauvaise interface.

**Binder le service à enp0s8 :**

```bash
sudo sed -i 's/^INTERFACESv4=.*/INTERFACESv4="enp0s8"/' /etc/default/isc-dhcp-server
```

> Si le nom diffère (ex. `eth1`), adapte-le ici et partout ensuite.

## 1.2 Configurer la plage DHCP

```bash
sudo tee /etc/dhcp/dhcpd.conf >/dev/null <<'EOF'
option domain-name "lab.local";
option domain-name-servers 8.8.8.8, 1.1.1.1;

default-lease-time 600;      # 10 min
max-lease-time 7200;         # 2 h
authoritative;               # "je suis l'autorité" sur ce segment

subnet 10.10.10.0 netmask 255.255.255.0 {
  range 10.10.10.100 10.10.10.200;      # évite .11/.12/.13 posées par Vagrant
  option broadcast-address 10.10.10.255;
  # Pas d'option 'routers' ici pour l'instant (pas de sortie Internet via vm1).
}
EOF
```

**Pourquoi ces choix ?**

* **Plage 100–200** : on **évite les collisions** avec les IP statiques actuelles (.11/.12/.13).
* **`authoritative;`** : le serveur répond vite aux clients “perdus” (meilleure démo).
* **DNS publics** : simple pour tester la résolution (`dig`, `curl`).
* Pas de **`option routers`** tant que vm1 n’est pas routeur/NAT (on l’ajoutera plus tard si tu veux).

## 1.3 Démarrer et valider

```bash
sudo systemctl enable isc-dhcp-server
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server --no-pager
journalctl -u isc-dhcp-server -n 50 --no-pager
```

**À savoir / erreurs fréquentes**

* *“Not configured to listen on any interfaces”* → `INTERFACESv4` mal réglé.
* *“No subnet declaration for enp0s8”* → `/etc/dhcp/dhcpd.conf` ne contient pas `subnet 10.10.10.0/24`.
* *Refuse de démarrer* → vérifie que **vm1** a **une IP statique** sur `enp0s8` (posée par Vagrant), sinon déclare-la.

---

# Étape 2 — vm2 & vm3 : passer l’interface lab en DHCP (netplan)

> Vagrant a mis une IP **statique** via netplan (souvent `/etc/netplan/50-vagrant.yaml`).
> On **désactive ce fichier** et on crée un petit YAML **DHCP** pour `enp0s8`.

## 2.1 vm2

```bash
vagrant ssh vm2
sudo mkdir -p /root/netplan-backup && sudo cp -a /etc/netplan/* /root/netplan-backup/

ls -l /etc/netplan
sudo mv /etc/netplan/50-vagrant.yaml /etc/netplan/50-vagrant.yaml.disabled 2>/dev/null || true

sudo tee /etc/netplan/02-dhcp-lab.yaml >/dev/null <<'EOF'
network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: true
EOF

sudo netplan apply
sudo dhclient -r enp0s8 || true
sudo dhclient enp0s8 || true
ip -4 addr show enp0s8
```

**Pourquoi ces gestes ?**

* **Renommer** le YAML Vagrant : netplan **charge tous** les .yaml ; on évite d’avoir **2 configs contradictoires**.
* Nouveau fichier minimal : **on n’altère pas la carte NAT** (enp0s3).
* `netplan apply` applique tout de suite ; `dhclient` force un **renouvellement**.

## 2.2 vm3

… mêmes commandes que pour vm2.

**Attendu**

* vm2/vm3 reçoivent des IP entre **10.10.10.100–200** (selon DORA).
* Tu peux lire les baux sur vm1 :

  ```
  sudo tail -n +1 /var/lib/dhcp/dhcpd.leases
  ```

---

# Étape 3 — (Optionnel) Réservations par MAC (IP fixes via DHCP)

**Pourquoi ?**

* Pour un **TP reproductible**, attribue **toujours la même IP** à chaque VM **sans** config statique côté client.

1. Récupère la MAC d’enp0s8 sur vm2/vm3 :

```bash
ip link show enp0s8 | awk '/link\/ether/{print $2}'
```

2. Sur vm1, dans `/etc/dhcp/dhcpd.conf` :

```conf
host vm2 {
  hardware ethernet 08:00:27:aa:bb:cc;
  fixed-address 10.10.10.20;
}
host vm3 {
  hardware ethernet 08:00:27:dd:ee:ff;
  fixed-address 10.10.10.30;
}
```

3. Redémarre et renouvelle :

```bash
sudo systemctl restart isc-dhcp-server
# sur vm2/vm3
sudo dhclient -r enp0s8 && sudo dhclient enp0s8
```

---

# Étape 4 — (Option) Donner Internet à vm2/vm3 via vm1

> Ici on **ajoute** une passerelle par défaut côté clients **et** on transforme vm1 en **routeur/NAT**.

## 4.1 DHCP : annoncer la passerelle (vm1)

Sur vm1, dans `subnet { ... }` :

```conf
option routers 10.10.10.11;
```

Redémarre le service DHCP et renouvelle vm2/vm3.

**Pourquoi ?**

* Les clients vont installer une **route par défaut** `via 10.10.10.11`.

## 4.2 Activer le routage IP + NAT sur vm1

```bash
# forward L3
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-lab.conf
sudo sysctl -p /etc/sysctl.d/99-lab.conf

# NAT (iptables) vers Internet via enp0s3 (NAT VirtualBox)
sudo apt-get install -y iptables
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

# (persistance facultative)
# sudo apt-get install -y iptables-persistent
# sudo netfilter-persistent save
```

**Pourquoi ?**

* `ip_forward=1` : vm1 **route** entre 10.10.10.0/24 (LAN) et sa carte NAT.
* **MASQUERADE** : vm1 traduit les IP du LAN pour **sortir sur Internet** via sa NAT.

## 4.3 Tests

Sur vm2 :

```bash
ip route | grep default         # doit pointer vers 10.10.10.11
ping -c2 1.1.1.1                # OK
dig openai.com +short           # résolution DNS (grâce aux DNS fournis par DHCP)
traceroute 1.1.1.1              # voit vm1 comme 1er saut
```

---

# Étape 5 — Démos “réseau qui vit”

## 5.1 Observer DHCP en direct (DORA)

Sur vm1 :

```bash
sudo apt-get install -y tcpdump
sudo tcpdump -i enp0s8 -n port 67 or port 68
```

Puis, sur vm2 :

```bash
sudo dhclient -r enp0s8 && sudo dhclient enp0s8
```

**Tu verras** : **D**iscover → **O**ffer → **R**equest → **A**ck.

## 5.2 ARP côté clients

Sur vm2 :

```bash
ip neigh show
```

Tu dois voir l’IP de vm1 (10.10.10.11) associée à une **MAC** (si `option routers` a été utilisé et que tu as fait du trafic).

---

# Dépannage rapide

* **vm2/vm3 n’obtiennent pas d’IP**

  * `sudo systemctl status isc-dhcp-server` sur vm1 (erreurs).
  * `INTERFACESv4="enp0s8"` correct ?
  * `subnet 10.10.10.0/24` bien présent ?
  * Renommer **toutes** les anciennes configs netplan statiques sur vm2/vm3.

* **Deux serveurs DHCP concurrents**

  * Re-vérifie `VBoxManage list dhcpservers` sur l’hôte.
  * **Un seul** serveur DHCP doit répondre sur le segment lab.

* **Internet KO pour vm2/vm3 après Étape 4**

  * `ip route` sur vm2 : default via 10.10.10.11 ?
  * vm1 : `sysctl net.ipv4.ip_forward` = 1 ?
  * iptables MASQUERADE présent ? `sudo iptables -t nat -S | grep MASQUERADE`

---

# Revenir en arrière (rollback)

* **vm2/vm3** :

  ```bash
  sudo rm /etc/netplan/02-dhcp-lab.yaml
  sudo mv /etc/netplan/50-vagrant.yaml.disabled /etc/netplan/50-vagrant.yaml
  sudo netplan apply
  ```
* **vm1** :

  ```bash
  sudo systemctl stop isc-dhcp-server
  sudo apt-get purge -y isc-dhcp-server
  sudo rm -f /etc/dhcp/dhcpd.conf /etc/sysctl.d/99-lab.conf
  sudo iptables -t nat -F
  ```

---

Si tu veux, je peux te fournir **2 scripts prêts à lancer** :

* `setup-dhcp-on-vm1.sh` (installe/paramètre isc-dhcp-server + NAT optionnel),
* `switch-to-dhcp.sh` pour vm2/vm3 (sauvegarde netplan, active DHCP, renew).
