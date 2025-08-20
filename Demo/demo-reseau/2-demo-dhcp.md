Top, on va “bétonner” l’installation DHCP dans **ton lab 3 VMs** (Ubuntu) avec :

* les **étapes exactes**,
* le **pourquoi** derrière chaque réglage,
* des **contrôles** (logs, DORA, leases),
* des **options avancées** (NAT, réservations),
* et 2 **scripts prêts à l’emploi**.

> Contexte réseau (classique Vagrant/VirtualBox)
>
> * **vm1** : 10.10.10.11 — deviendra **serveur DHCP** (interface lab = `enp0s8`, NAT = `enp0s3`)
> * **vm2** : 10.10.10.12 — client DHCP
> * **vm3** : 10.10.10.13 — client DHCP
> * LAN du lab : **10.10.10.0/24** (host-only). NAT donne Internet à **vm1** (et, si tu veux, via vm1 aux autres)

---

# 0) Pré-checks côté hôte (VirtualBox)

*(évite un 2ᵉ DHCP qui perturberait le lab)*

```bash
# Sur l'hôte
VBoxManage list hostonlyifs
VBoxManage list dhcpservers
```

**Pourquoi ?** S’assurer qu’**aucun** DHCP VirtualBox n’est actif sur le réseau host-only du lab. Un seul serveur DHCP doit répondre sur un segment.

---

# 1) Installer et **lier** isc-dhcp-server à la bonne interface (vm1)

```bash
vagrant ssh vm1
sudo apt update
sudo apt install -y isc-dhcp-server

# Identifier l'interface du lab
ip -o -4 addr show | awk '{print $2,$4}'
# Attendu : enp0s8 a 10.10.10.11/24

# Lier le serveur DHCP à enp0s8 (important)
sudo sed -i 's/^INTERFACESv4=.*/INTERFACESv4="enp0s8"/' /etc/default/isc-dhcp-server
```

**Pourquoi ?**

* `INTERFACESv4="enp0s8"` oblige le service à **écouter seulement** sur l’interface du lab (et pas sur la carte NAT), pour éviter d’annoncer par erreur sur d’autres segments.
* S’il n’est lié à aucune interface valide, le service **refuse de démarrer**.

---

# 2) Configurer la plage DHCP (vm1)

Fichier **/etc/dhcp/dhcpd.conf** annoté :

```conf
# --- Options globales ---
option domain-name "lab.local";          # Contexte DNS logique (facultatif)
option domain-name-servers 8.8.8.8, 1.1.1.1;  # DNS fournis aux clients (tests faciles)
default-lease-time 600;                  # 10 min : démos rapides (renouvellements visibles)
max-lease-time 7200;                     # 2 h : plafond raisonnable
authoritative;                           # "Je suis la référence sur ce LAN" (réponses plus rapides aux clients perdus)

# --- Déclaration du sous-réseau du lab ---
subnet 10.10.10.0 netmask 255.255.255.0 {
  range 10.10.10.100 10.10.10.200;       # Évite .11/.12/.13 déjà prises (vm1/2/3 avant bascule)
  option broadcast-address 10.10.10.255;
  # Pas d'option routers pour l’instant (pas de sortie Internet via vm1)
  # option routers 10.10.10.11;          # (on l'activera plus tard si vm1 sert aussi de passerelle)
}
```

**Pourquoi ces choix ?**

* **Plage 100–200** : pas de conflit avec les IP “historiques” posées par Vagrant.
* **`authoritative`** : accélère la convergence (les clients cessent d’attendre un autre serveur).
* **DNS publics** : tests `dig`/`curl` faciles.
* **Pas de gateway** tant que vm1 n’est pas configurée en routeur/NAT.

---

# 3) Démarrer et valider le service (vm1)

```bash
sudo systemctl enable isc-dhcp-server
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server --no-pager
journalctl -u isc-dhcp-server -n 50 --no-pager
```

**À comprendre / erreurs fréquentes**

* **“Not configured to listen on any interfaces”** → `INTERFACESv4` incorrect.
* **“No subnet declaration for enp0s8”** → oubli du bloc `subnet 10.10.10.0/24`.
* Si vm1 **n’a pas** d’IP sur `enp0s8`, déclare correctement le réseau ou fixe l’IP.

---

# 4) Bascule **vm2** et **vm3** en DHCP (netplan)

> Vagrant a mis une IP **statique** via netplan (`/etc/netplan/50-vagrant.yaml`).
> On la désactive et on crée un YAML minimal **dhcp4: true** pour `enp0s8`.

### vm2

```bash
vagrant ssh vm2
# Sauvegarde
sudo mkdir -p /root/netplan-backup && sudo cp -a /etc/netplan/* /root/netplan-backup/
# Désactiver le YAML statique Vagrant
sudo mv /etc/netplan/50-vagrant.yaml /etc/netplan/50-vagrant.yaml.disabled 2>/dev/null || true
# Nouveau YAML DHCP
sudo tee /etc/netplan/02-dhcp-lab.yaml >/dev/null <<'EOF'
network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: true
EOF
# Appliquer et forcer un bail
sudo netplan apply
sudo dhclient -r enp0s8 || true
sudo dhclient enp0s8 || true

ip -4 addr show enp0s8   # doit afficher une IP 10.10.10.100–200
```

### vm3

Même procédure.

**Pourquoi ça marche sans te couper la session ?**

* On ne touche **pas** à `enp0s3` (NAT), donc `vagrant ssh` (via NAT) reste OK.
* On change **uniquement** l’interface du **lab**.

---

# 5) Vérifier les baux et DORA (vm1)

### Leases

```bash
sudo tail -n +1 /var/lib/dhcp/dhcpd.leases
```

Tu y vois l’historique des baux : IP attribuée, MAC, bail, etc.

### DORA en live (pédago)

```bash
sudo apt install -y tcpdump
sudo tcpdump -ni enp0s8 -vv 'udp and (port 67 or port 68)'
```

Puis, sur vm2 :

```bash
sudo dhclient -r enp0s8 && sudo dhclient enp0s8
```

**À expliquer aux élèves**

* **Discover** (client → 255.255.255.255, UDP 68→67, IP source 0.0.0.0)
* **Offer** (serveur → 255.255.255.255 ou client)
* **Request** (client demande l’offre)
* **Ack** (serveur confirme)
  On voit bien les 4 étapes au sniff.

---

# 6) (Option) Réservations par MAC (IP fixes via DHCP)

**Pourquoi ?** Conserver des IP “stables” **sans** config statique côté client (utile en TP).

1. Récupérer la MAC (vm2/vm3) :

```bash
ip link show enp0s8 | awk '/link\/ether/ {print $2}'
```

2. Déclarer dans `/etc/dhcp/dhcpd.conf` (vm1) :

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

3. Redémarrer et renouveler :

```bash
sudo systemctl restart isc-dhcp-server
# sur vm2/vm3
sudo dhclient -r enp0s8 && sudo dhclient enp0s8
```

---

# 7) (Option) Donner Internet à vm2/vm3 via **vm1** (passerelle + NAT)

## 7.1 Pousser la gateway par DHCP (vm1)

Dans le bloc `subnet` :

```conf
option routers 10.10.10.11;
```

**Pourquoi ?** Les clients installent **default via 10.10.10.11** automatiquement.

## 7.2 Activer le routage + NAT sur vm1

```bash
# Activer le forward L3
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-lab.conf
sudo sysctl -p /etc/sysctl.d/99-lab.conf

# NAT vers Internet via la carte NAT de vm1 (enp0s3)
sudo apt-get install -y iptables
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

# (Option) rendre persistant
# sudo apt-get install -y iptables-persistent
# sudo netfilter-persistent save
```

## 7.3 Tests (vm2)

```bash
ip route | grep default        # doit pointer vers 10.10.10.11
ping -c2 1.1.1.1               # OK
dig debian.org +short          # OK (DNS fournis par DHCP)
traceroute 1.1.1.1             # vm1 en 1er saut
```

**Pourquoi ça marche ?** vm1 traduit (MASQUERADE) les IP du LAN en son IP NAT sortante.

---

# 8) Dépannage (symptôme → cause → commande → correctif)

* **vm2/vm3 n’obtiennent pas d’IP**

  * Cause : service DHCP KO, mauvaise interface, pas de bloc `subnet`.
  * Cmd : `systemctl status isc-dhcp-server`, `journalctl -u`, `cat /etc/default/isc-dhcp-server`.
  * Fix : `INTERFACESv4="enp0s8"`, corriger `dhcpd.conf`, redémarrer.

* **Deux serveurs répondent**

  * Cause : DHCP VirtualBox actif.
  * Cmd : `VBoxManage list dhcpservers`.
  * Fix : désactiver DHCP VirtualBox sur ce host-only.

* **Internet KO sur vm2 après étape 7**

  * Cause : pas de default route, forward désactivé, pas de MASQUERADE.
  * Cmd : `ip r | grep default`, `sysctl net.ipv4.ip_forward`, `iptables -t nat -S | grep MASQUERADE`.
  * Fix : pousser `option routers`, activer forward, ajouter règle NAT.

* **Leases bizarres / pas à jour**

  * Cause : ancien bail en cache côté client.
  * Cmd : `dhclient -r enp0s8 ; dhclient enp0s8`.
  * Fix : forcer un nouveau bail.

---

# 9) Rollback rapide

**vm2/vm3** (revenir à la config Vagrant statique) :

```bash
sudo rm -f /etc/netplan/02-dhcp-lab.yaml
sudo mv /etc/netplan/50-vagrant.yaml.disabled /etc/netplan/50-vagrant.yaml
sudo netplan apply
```

**vm1** :

```bash
sudo systemctl stop isc-dhcp-server
sudo apt purge -y isc-dhcp-server
sudo rm -f /etc/dhcp/dhcpd.conf /etc/sysctl.d/99-lab.conf
sudo iptables -t nat -F
```

---

## 🔧 Scripts prêts à l’emploi

### A) `setup-dhcp-on-vm1.sh` (vm1)

```bash
#!/usr/bin/env bash
set -euo pipefail

# Vars
LAB_IF="${LAB_IF:-enp0s8}"           # interface du lab
NAT_IF="${NAT_IF:-enp0s3}"           # interface NAT (pour l'option NAT)
SUBNET="${SUBNET:-10.10.10.0}"
NETMASK="${NETMASK:-255.255.255.0}"
RANGE_START="${RANGE_START:-10.10.10.100}"
RANGE_END="${RANGE_END:-10.10.10.200}"
DNS1="${DNS1:-8.8.8.8}"
DNS2="${DNS2:-1.1.1.1}"
DOMAIN="${DOMAIN:-lab.local}"
ROUTER_OPT="${ROUTER_OPT:-no}"       # yes pour pousser option routers
ENABLE_NAT="${ENABLE_NAT:-no}"       # yes pour activer ip_forward + MASQUERADE

need() { command -v "$1" >/dev/null 2>&1 || { echo "Missing $1"; exit 1; }; }

echo "[*] Installing isc-dhcp-server..."
sudo apt-get update -y
sudo apt-get install -y isc-dhcp-server iproute2

echo "[*] Checking $LAB_IF has an IP in $SUBNET ..."
ip -4 addr show "$LAB_IF" || { echo "Interface $LAB_IF not found"; exit 1; }

echo "[*] Binding DHCP to $LAB_IF ..."
sudo sed -i "s/^INTERFACESv4=.*/INTERFACESv4=\"$LAB_IF\"/" /etc/default/isc-dhcp-server

ROUTERS_LINE=""
if [[ "$ROUTER_OPT" == "yes" ]]; then
  # Déduire l'IP de la passerelle depuis l'IP de $LAB_IF si possible
  GATEWAY_IP=$(ip -4 -o addr show "$LAB_IF" | awk '{print $4}' | cut -d/ -f1)
  ROUTERS_LINE="  option routers $GATEWAY_IP;"
  echo "[*] Will advertise router: $GATEWAY_IP"
fi

echo "[*] Writing /etc/dhcp/dhcpd.conf ..."
sudo tee /etc/dhcp/dhcpd.conf >/dev/null <<EOF
option domain-name "$DOMAIN";
option domain-name-servers $DNS1, $DNS2;

default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet $SUBNET netmask $NETMASK {
  range $RANGE_START $RANGE_END;
  option broadcast-address $(ipcalc -b "$SUBNET/$NETMASK" 2>/dev/null | awk '/Broadcast/ {print $2}');
$ROUTERS_LINE
}
EOF

echo "[*] Enabling & restarting DHCP server ..."
sudo systemctl enable isc-dhcp-server
sudo systemctl restart isc-dhcp-server
sudo systemctl --no-pager --full status isc-dhcp-server || true
journalctl -u isc-dhcp-server -n 20 --no-pager || true

if [[ "$ENABLE_NAT" == "yes" ]]; then
  echo "[*] Enabling IP forwarding + NAT via $NAT_IF ..."
  echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-lab.conf
  sudo sysctl -p /etc/sysctl.d/99-lab.conf
  sudo apt-get install -y iptables
  sudo iptables -t nat -C POSTROUTING -o "$NAT_IF" -j MASQUERADE 2>/dev/null || \
    sudo iptables -t nat -A POSTROUTING -o "$NAT_IF" -j MASQUERADE
  # Optional persistence:
  # sudo apt-get install -y iptables-persistent && sudo netfilter-persistent save
fi

echo "[*] Done. Check leases: sudo tail -n +1 /var/lib/dhcp/dhcpd.leases"
```

**Usage** (sur vm1) :

```bash
# simple
bash setup-dhcp-on-vm1.sh

# avec passerelle annoncée + NAT
ROUTER_OPT=yes ENABLE_NAT=yes bash setup-dhcp-on-vm1.sh
```

---

### B) `switch-to-dhcp.sh` (vm2/vm3)

```bash
#!/usr/bin/env bash
set -euo pipefail

IFACE="${IFACE:-enp0s8}"

echo "[*] Backing up /etc/netplan -> /root/netplan-backup"
sudo mkdir -p /root/netplan-backup
sudo cp -a /etc/netplan/* /root/netplan-backup/ || true

echo "[*] Disabling Vagrant static YAML if present ..."
if ls /etc/netplan/*vagrant*.yaml >/dev/null 2>&1; then
  for f in /etc/netplan/*vagrant*.yaml; do
    sudo mv "$f" "$f.disabled"
  done
fi

echo "[*] Writing DHCP YAML for $IFACE ..."
sudo tee /etc/netplan/02-dhcp-lab.yaml >/dev/null <<EOF
network:
  version: 2
  ethernets:
    $IFACE:
      dhcp4: true
EOF

echo "[*] Applying netplan + renewing DHCP ..."
sudo netplan apply
sudo dhclient -r "$IFACE" || true
sudo dhclient "$IFACE" || true

echo "[*] Current address:"
ip -4 addr show "$IFACE" | awk '/inet /{print $2}'

echo "[*] Default route:"
ip route | awk '/^default/{print}'
```

**Usage** :

```bash
bash switch-to-dhcp.sh
```

---

## Pourquoi ces choix pédagogiques fonctionnent bien

* Tu **maîtrises** qui parle DHCP (binding d’interface) et **où** (subnet déclaré).
* Tu peux **montrer DORA** en live (tcpdump) et **lire les baux** (`dhcpd.leases`).
* Tu sépares **services d’infra** (DHCP) et **routage/NAT** (optionnel), pour illustrer les rôles.
* Les **réservations MAC** rendent tes TPs **reproductibles** sans toucher aux clients.

Si tu veux, je te prépare un **énoncé TP “pas à pas”** (avec captures attendues de `tcpdump`, `journalctl`, `ip route`, etc.) que tes apprenants pourront suivre en autonomie.
