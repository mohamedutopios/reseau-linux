Super 👍 tu es tombé au bon endroit :

Le répertoire **`/run/systemd/netif/leases/`** contient les fichiers créés par **systemd-networkd** ou **systemd-resolved** lors de l’attribution DHCP.

Dans ton cas, il y a un fichier `2` (parce que c’est l’**index du lien réseau**).

---

## 🔎 Vérifie son contenu

```bash
cat /run/systemd/netif/leases/2
```

Tu vas voir quelque chose comme :

```
ADDRESS=10.0.2.15
NETMASK=255.255.255.0
ROUTER=10.0.2.2
SERVER_ADDRESS=10.0.2.2
DNS=10.0.2.3
NTP=10.0.2.3
DOMAINNAME=...
```

### Explication des champs :

* **ADDRESS** → l’IP donnée à la VM (`10.0.2.15`)
* **NETMASK** → masque du réseau NAT (`255.255.255.0`)
* **ROUTER** → la passerelle (toujours `10.0.2.2` en VirtualBox NAT)
* **DNS** → le proxy DNS (`10.0.2.3`)
* **SERVER\_ADDRESS** → l’adresse du serveur DHCP (souvent `10.0.2.2`)
* **NTP** → parfois aussi `10.0.2.3` (si VirtualBox fournit un proxy NTP)

---

## 🔧 Comment ça marche

1. **Ubuntu** démarre → `systemd-networkd` demande une adresse IP en **DHCP** sur l’interface `enp0s3`.
2. **VirtualBox NAT DHCP** répond → IP 10.0.2.15, gateway 10.0.2.2, DNS 10.0.2.3.
3. Ubuntu stocke ce bail dans `/run/systemd/netif/leases/…`.
4. C’est ce bail qui remplit la **table de routage** (`ip route`) et \*\*/etc/resolv.conf\` (DNS).

---

👉 Tu veux que je te montre aussi **comment relier ce fichier `2` à l’interface réseau correspondante** (ex. savoir si c’est enp0s3 ou enp0s8) avec la commande `networkctl` ?
