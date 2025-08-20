Netplan est **le gestionnaire de configuration réseau “haut-niveau”** d’Ubuntu (et dérivés récents).
Tu écris **un ou plusieurs fichiers YAML** dans `/etc/netplan/*.yaml` ; Netplan **traduit** ensuite ces YAML vers un **moteur bas-niveau** (“backend”) :

* **systemd-networkd** (serveurs, minimal, par défaut sur Ubuntu Server)
* **NetworkManager** (postes desktop)

Puis il applique la config au système.

---

## Comment ça marche (vue rapide)

1. Tu décris le réseau en YAML.
2. `netplan generate` **compile** en fichiers pour le backend (dans `/run/systemd/network/` ou pour NM).
3. `netplan apply` **active** la config (up/down, DHCP, routes, DNS…).
4. `netplan try` applique **temporairement** (revert auto si tu ne valides pas → évite de te couper l’accès SSH).

Commandes utiles :

```bash
netplan get               # montre la config effective (fusionnée)
sudo netplan generate     # (re)génère les fichiers du backend
sudo netplan apply        # applique
sudo netplan try          # applique avec "filet de sécurité"
sudo netplan apply --debug  # débogage verbeux
```

---

## Anatomie d’un fichier Netplan

Emplacement : `/etc/netplan/<ordre>-<nom>.yaml` (chargés **par ordre lexicographique**).

Structure minimale :

```yaml
network:
  version: 2
  renderer: networkd        # ou "NetworkManager"
  ethernets:
    enp0s8:
      dhcp4: true           # DHCP IPv4
```

Champs courants :

* `addresses: [10.10.10.11/24]` (IP(s) statiques)
* `gateway4: 10.10.10.1` (route par défaut)
* `nameservers: { addresses: [8.8.8.8,1.1.1.1], search: [lab.local] }`
* `routes:` (routes statiques)
* `match:` (associer une config à une interface via `macaddress:` ou `name:`)
* Sections possibles : `ethernets`, `wifis`, `bridges`, `vlans`, `bonds`…

> Attention à l’**indentation** (espaces, pas de tabulations) : YAML est strict.

---

## Exemples prêts pour ton lab 3 VMs (réseau 10.10.10.0/24)

### 1) Client en **DHCP** sur `enp0s8` (vm2/vm3)

```yaml
# /etc/netplan/02-dhcp-lab.yaml
network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: true
```

Appliquer :

```bash
sudo netplan apply
```

### 2) Serveur avec **IP statique** (vm1)

```yaml
# /etc/netplan/01-static-lab.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      addresses: [10.10.10.11/24]
      nameservers:
        addresses: [8.8.8.8,1.1.1.1]
      # gateway4: 10.10.10.1        # ajoute si vm1 doit sortir vers Internet
```

### 3) Ajouter une **route statique**

```yaml
routes:
  - to: 192.168.50.0/24
    via: 10.10.10.13
```

---

## Débogage & interactions

* Backend **systemd-networkd** :
  `networkctl`, `networkctl status enp0s8`, `journalctl -u systemd-networkd`
* Backend **NetworkManager** :
  `nmcli con show`, `nmcli dev show enp0s8`, logs `journalctl -u NetworkManager`
* **Plusieurs YAML** : ils sont fusionnés ; un ancien fichier (ex. `50-vagrant.yaml`, `50-cloud-init.yaml`) peut **écraser** ta config → **renomme-le .disabled** pour neutraliser.
* Si tu appliques à distance, préfère `sudo netplan try` (tu valides en 120 s, sinon rollback).

---

## Pourquoi Netplan ?

* **Déclaratif, lisible** (YAML) et **idempotent**.
* Abstrait le choix du backend (même fichier marche pour desktop/serveur).
* Gère d’office DHCP, IP statiques, routes, DNS, VLAN/bridge/bond.
* S’intègre avec **cloud-init** (souvent un `50-cloud-init.yaml` initial).

---

## Petits pièges fréquents (et solutions)

* **Rien ne change après apply** → il reste un autre YAML prioritaire (`ls /etc/netplan`).
* **Perte SSH après modif** → utiliser `netplan try`.
* **Nom d’interface** faux (ex. `eth0` au lieu de `enp0s8`) → `ip link` pour vérifier.
* **Booleans** YAML : `true/false` (pas `"true"` entre guillemets).
* **Renderer** : ne mets pas `NetworkManager` sur un serveur sans NM installé.

---

Si tu veux, je peux te relire ton/tes fichiers `/etc/netplan/*.yaml` (colle-les ici) et te dire **exactement** ce qui se passera après `netplan apply`, avec les commandes de vérification adaptées.
