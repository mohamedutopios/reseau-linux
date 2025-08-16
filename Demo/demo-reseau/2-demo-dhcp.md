Oui, c’est possible **sans toucher au Vagrantfile**. On va juste agir **à l’intérieur des VMs** :

* **vm1** → deviendra **serveur DHCP** (isc-dhcp-server) sur l’interface du lab (enp0s8).
* **vm2** et **vm3** → passeront de l’IP statique (posée par Vagrant) à **DHCP** en modifiant netplan **dans la VM**.
* ⚠️ Évite ensuite `vagrant reload`/`destroy` (Vagrant remettrait l’IP statique du Vagrantfile). Garde les VMs allumées pour le lab.

---

# Étape 0 — Vérifs rapides (sur l’hôte)

Par défaut, le réseau `private_network` créé par Vagrant **n’a pas** de DHCP VirtualBox. Tu peux vérifier (optionnel) :

```bash
# Sur l'hôte
VBoxManage list hostonlyifs
VBoxManage list dhcpservers
```

S’il existe un DHCP VirtualBox pour ce réseau, **désactive-le** (rare avec private\_network).

---

# Étape 1 — vm1 en serveur DHCP

Sur **vm1** :

```bash
vagrant ssh vm1
sudo apt-get update
sudo apt-get install -y isc-dhcp-server
```

Identifier l’interface du lab (souvent `enp0s8`) :

```bash
ip -o link show | awk -F': ' '{print $2}'
ip -4 addr show enp0s8
```

Configurer l’interface d’écoute DHCP :

```bash
sudo sed -i 's/^INTERFACESv4=.*/INTERFACESv4="enp0s8"/' /etc/default/isc-dhcp-server
```

Configurer le **scope** `/etc/dhcp/dhcpd.conf` (simple, sans passerelle pour l’instant) :

```bash
sudo tee /etc/dhcp/dhcpd.conf >/dev/null <<'EOF'
option domain-name "lab.local";
option domain-name-servers 8.8.8.8, 1.1.1.1;

default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 10.10.10.0 netmask 255.255.255.0 {
  range 10.10.10.100 10.10.10.200;
  option broadcast-address 10.10.10.255;

  # Pas d'option routers pour l'instant (on ne route pas vers Internet via vm1).
  # Tu pourras l'ajouter plus tard si tu fais de vm1 un routeur/NAT.
}
EOF
```

Démarrer :

```bash
sudo systemctl enable isc-dhcp-server
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server --no-pager
journalctl -u isc-dhcp-server -n 50 --no-pager
```

> Si le service se plaint d’une interface, re-vérifie le nom (`enp0s8`) et que vm1 a bien une IP dans 10.10.10.0/24 sur cette interface (Vagrant l’a posée).

---

# Étape 2 — Basculer vm2 et vm3 en DHCP (à l’intérieur des VMs)

Par défaut, Vagrant a posé une config **statique** via netplan (souvent dans `/etc/netplan/50-vagrant.yaml`).
On va **désactiver** cette config et mettre **dhcp4: true**.

## Sur vm2

```bash
vagrant ssh vm2

# Sauvegarder les configs netplan
sudo mkdir -p /root/netplan-backup
sudo cp -a /etc/netplan/* /root/netplan-backup/

# Désactiver la config IP statique de Vagrant (renommer le fichier)
# (Si le nom exact diffère, liste /etc/netplan pour le trouver)
ls -l /etc/netplan
sudo mv /etc/netplan/50-vagrant.yaml /etc/netplan/50-vagrant.yaml.disabled 2>/dev/null || true

# Créer une config DHCP pour enp0s8
sudo tee /etc/netplan/02-dhcp-lab.yaml >/dev/null <<'EOF'
network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: true
EOF

# Appliquer
sudo netplan apply

# Forcer un renew si besoin
sudo dhclient -r enp0s8 || true
sudo dhclient enp0s8 || true

# Vérifier l'IP obtenue
ip -4 addr show enp0s8
```

## Sur vm3

Même chose :

```bash
vagrant ssh vm3
sudo mkdir -p /root/netplan-backup
sudo cp -a /etc/netplan/* /root/netplan-backup/
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

> Attendu : **vm2** et **vm3** reçoivent une IP dans **10.10.10.100–200**.
> Tu peux regarder les baux sur vm1 : `/var/lib/dhcp/dhcpd.leases`.

---

# Étape 3 — (Optionnel) Réservations DHCP par MAC

Récupère les **MAC** d’enp0s8 sur vm2 et vm3 :

```bash
ip link show enp0s8 | awk '/link\/ether/{print $2}'
```

Ajoute-les dans `/etc/dhcp/dhcpd.conf` sur **vm1** :

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

Puis :

```bash
sudo systemctl restart isc-dhcp-server
```

Renouvelle sur vm2/vm3 :

```bash
sudo dhclient -r enp0s8 && sudo dhclient enp0s8
```

---

# Tests utiles

Sur vm2 / vm3 :

```bash
ip -4 addr show enp0s8
ping -c2 10.10.10.11        # ping du serveur DHCP (vm1)
ping -c2 10.10.10.100       # ping pair (si attribué)
```

DNS (ici via Internet public, car on a mis 8.8.8.8/1.1.1.1 dans le DHCP) :

```bash
dig openai.com
curl -I https://openai.com
```

---

## Remarques importantes

* **Ne pas faire** `vagrant reload`/`destroy` pendant le TP (Vagrant remettrait l’IP statique définie dans le Vagrantfile).
* Si tu veux plus tard que **vm1 serve de passerelle vers Internet** pour vm2/vm3, ajoute dans le DHCP :

  ```conf
  option routers 10.10.10.11;
  ```

  et sur **vm1** active le **forwarding** + NAT (iptables/nftables). Je peux te donner le script prêt-à-l’emploi si tu veux.

Souhaites-tu que je te génère un **script auto** pour transformer vm1 en DHCP (et un autre pour basculer vm2/vm3 en DHCP), afin que tu n’aies qu’à lancer 2 commandes ?
