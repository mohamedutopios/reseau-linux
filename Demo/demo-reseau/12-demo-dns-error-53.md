Le message est clair : **le port 53 est déjà utilisé**, donc `dnsmasq` n’arrive pas à écouter et plante.

Voici la marche rapide pour diagnostiquer puis corriger, avec 3 options selon ton objectif.

---

## 1) Diagnostic éclair

```bash
# Qui occupe le port 53 (UDP/TCP) ?
sudo ss -lunpt | grep ':53'
sudo lsof -iUDP:53 -iTCP:53 -sTCP:LISTEN

# Si tu utilises systemd-resolved (Ubuntu par défaut)
resolvectl status
systemctl status systemd-resolved --no-pager
```

Les coupables habituels : **systemd-resolved**, **bind9/named**, un **autre dnsmasq** (ex. via NetworkManager), ou un conteneur en host-network.

---

## 2) Corrections possibles (choisis UNE des options)

### Option A — Tu veux **dnsmasq pour DHCP seulement** (pas DNS)

C’est le plus simple pour enlever le conflit.

1. Crée un fichier dédié :

```bash
sudo tee /etc/dnsmasq.d/dhcp-only.conf >/dev/null <<'EOF'
# Désactive le serveur DNS de dnsmasq -> pas d'écoute sur le port 53
port=0

# N'écoute que sur l'interface du lab (adapte si besoin)
interface=enp0s8
bind-interfaces

# DHCP du lab (exemple 10.10.10.0/24)
dhcp-range=10.10.10.50,10.10.10.100,255.255.255.0,12h

# Option passerelle (mets l’IP de ta gateway si tu en as une)
# dhcp-option=3,10.10.10.1

# DNS proposés aux clients (si tu veux pousser des DNS publics)
# dhcp-option=6,1.1.1.1,8.8.8.8
EOF
```

2. Redémarre :

```bash
sudo systemctl restart dnsmasq
sudo systemctl status dnsmasq --no-pager
```

> Comme `port=0`, plus aucun conflit sur 53.

---

### Option B — Tu veux **dnsmasq faire DNS + DHCP**, en cohabitant avec `systemd-resolved`

Fais écouter `dnsmasq` **uniquement** sur l’IP du lab (pas sur loopback), pour éviter la collision avec `127.0.0.53:53`.

```bash
sudo tee /etc/dnsmasq.d/lab-dns-dhcp.conf >/dev/null <<'EOF'
# N'écoute que sur l'IP du lab
interface=enp0s8
bind-interfaces
listen-address=10.10.10.12
# (remplace 10.10.10.12 par l'IP de vm2 sur le lab)

# DHCP
dhcp-range=10.10.10.50,10.10.10.100,255.255.255.0,12h
# dhcp-option=3,10.10.10.1   # passerelle si nécessaire

# Résolution DNS : ne pas lire /etc/resolv.conf, définis des upstream
no-resolv
server=1.1.1.1
server=8.8.8.8
EOF

sudo systemctl restart dnsmasq
sudo systemctl status dnsmasq --no-pager
```

**Vérifie** que `dnsmasq` écoute bien **seulement** sur l’IP du lab :

```bash
sudo ss -lunpt | grep ':53'
```

---

### Option C — Tu veux **dnsmasq comme DNS principal du système** (remplacer `systemd-resolved`)

1. Désactiver l’écoute stub de `systemd-resolved` :

```bash
sudo sed -i 's/^#\?DNSStubListener=.*/DNSStubListener=no/' /etc/systemd/resolved.conf
```

2. Reconfigurer `resolv.conf` (pour qu’il n’utilise plus 127.0.0.53) :

```bash
# Option la plus simple : pointer le système vers dnsmasq local
echo 'nameserver 127.0.0.1' | sudo tee /etc/resolv.conf
# (ou vers l’IP lab de dnsmasq si tu préfères)
```

3. Redémarrer les services :

```bash
sudo systemctl restart systemd-resolved
sudo systemctl restart dnsmasq
sudo systemctl status dnsmasq --no-pager
```

> Si tu as **bind9** installé : `sudo systemctl disable --now bind9` (ou `named`), voire `sudo apt purge bind9` si inutile.

---

## 3) Tests rapides

```bash
# Côté serveur (vm2)
journalctl -u dnsmasq -n 50 --no-pager
sudo ss -lunpt | grep ':53'

# Test DNS depuis un client du lab (vm1/vm3)
dig @10.10.10.12 example.com
# ou
nslookup example.com 10.10.10.12

# Test DHCP (sur un client)
sudo dhclient -r enp0s8 && sudo dhclient enp0s8
ip a show enp0s8
```

---

## Astuces fréquentes

* Si tu utilises **NetworkManager**, vérifie qu’il n’active pas un **dnsmasq embarqué** :

  ```bash
  ps aux | grep dnsmasq
  ```

  (S’il y a un autre `dnsmasq` qui écoute, désactive la fonctionnalité DNSMasq de NM ou arrête NM sur un serveur.)

* Si tu tournes des **conteneurs** avec du host-network sur 53, change leur port ou leur réseau.

* Sur Ubuntu Server, l’interface du réseau privé Vagrant est souvent **`enp0s8`** (comme dans tes derniers logs). Adapte si besoin avec `ip -br a`.

---

### En résumé

* Le plus rapide pour repartir : **Option A** (`port=0`) si tu n’as besoin que de **DHCP**.
* Si tu veux **DNS + DHCP** sans toucher à `systemd-resolved` : **Option B** (écoute uniquement sur l’IP du lab).
* Si tu veux que **dnsmasq remplace le DNS système** : **Option C** (désactiver le stub de `resolved`).

Tu me dis quelle option tu choisis et je te donne un fichier de conf final prêt-à-coller adapté à **tes IPs exactes** et à **ta passerelle**.
