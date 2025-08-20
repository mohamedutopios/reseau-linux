Super question 👌 — sur **ISC DHCP (Ubuntu/Debian)** il y a 3 fichiers “cœur” côté **serveur**, et 2 fichiers utiles côté **client**.
Je te dis **où ils sont**, **à quoi ils servent**, et **de quoi ils sont composés** (avec mini-exemples annotés).

---

# 1) Serveur DHCP (vm1)

## A. **/etc/dhcp/dhcpd.conf** — *la* configuration du serveur

**Rôle :** décrire ce que le serveur annonce : options globales, sous-réseaux, plages d’adresses, réservations, politiques.

**Structure (grammaire) :**

* Fin de ligne par **`;`**.
* Blocs entre **`{ ... }`** (ex. `subnet {}`, `pool {}`, `host {}`).
* **Commentaires** : `# ...`
* Possibilité de **factoriser** : `include "/etc/dhcp/mon-fichier.inc";`

**Contenu typique (annoté) :**

```conf
# --- Global ---
authoritative;                          # je suis l'autorité sur ce LAN
default-lease-time 600;                 # bail par défaut (secondes)
max-lease-time 7200;                    # bail max (secondes)
ddns-update-style none;                 # pas de mises à jour DNS dynamiques

option domain-name "lab.local";         # nom de domaine logique
option domain-name-servers 8.8.8.8, 1.1.1.1;   # DNS poussés aux clients

# --- Sous-réseau du lab ---
subnet 10.10.10.0 netmask 255.255.255.0 {
  option broadcast-address 10.10.10.255;
  # option routers 10.10.10.11;         # passerelle (si vm1 route/NAT)

  # Plage dynamique (pool) : qui peut recevoir une IP ?
  pool {
    range 10.10.10.100 10.10.10.200;    # baux dynamiques (évite .11/.12/.13)
    allow unknown-clients;              # on sert aussi les clients non-réservés
  }
}

# --- Réservations (IP fixes via MAC) ---
host vm2 {
  hardware ethernet 08:00:27:aa:bb:cc;  # MAC enp0s8 de vm2
  fixed-address 10.10.10.20;            # IP réservée (souvent hors de la plage)
}
host vm3 {
  hardware ethernet 08:00:27:dd:ee:ff;
  fixed-address 10.10.10.30;
}

# --- (Exemples avancés éventuels) ---
# next-server 10.10.10.11; filename "pxelinux.0";   # PXE/TFTP
# class "telephones" { match if substring (option vendor-class-identifier,0,10) = "IPPhone"; }
# pool { range 10.10.10.150 10.10.10.160; allow members of "telephones"; }
```

> À retenir :
>
> * **Global** (options, durées, `authoritative`)
> * **`subnet {}`** (masque, broadcast, **pool/range** et éventuellement **gateway**)
> * **`host {}`** (réservations par **MAC** avec `fixed-address`)
> * **`pool {}`** pour affiner qui a droit à quelle plage (allow/deny)

---

## B. **/etc/default/isc-dhcp-server** — liaison aux interfaces

**Rôle :** dire au service **sur quelles interfaces** écouter (spécifique Debian/Ubuntu).

**Contenu typique :**

```bash
INTERFACESv4="enp0s8"   # interface du lab (host-only)
INTERFACESv6=""         # vide si tu ne fais pas de DHCPv6
# OPTIONS="-4"          # paramètres supplémentaires au besoin
```

> Si `INTERFACESv4` n’est pas correct, le service **refuse de démarrer** (“Not configured to listen on any interfaces”).

---

## C. **/var/lib/dhcp/dhcpd.leases** — base des baux (état runtime)

**Rôle :** mémoire des baux attribués (actifs/expirés). Indispensable pour diagnostiquer.

**À quoi ça ressemble :**

```conf
lease 10.10.10.101 {
  starts 4 2025/08/20 09:10:35;      # début de bail (UTC)
  ends   4 2025/08/20 09:20:35;      # fin de bail
  cltt   4 2025/08/20 09:10:35;      # client last transaction time
  binding state active;              # état (active/expired/free)
  hardware ethernet 08:00:27:12:34:56;
  uid "\001\010\000\000\000\001";    # (eventuel client-identifier)
  client-hostname "vm2";             # nom annoncé par le client
}
```

> Tu peux le **lire** (`tail`, `grep`), mais **ne l’édite pas** manuellement.

---

# 2) Côté client DHCP (vm2/vm3) — utile à connaître

## D. **/etc/dhcp/dhclient.conf** — préférences du client

**Rôle :** forcer ou ignorer certaines options côté client, envoyer un hostname, etc.

**Exemples utiles :**

```conf
send host-name "vm2";
# supersede domain-name-servers 10.10.10.11;   # forcer un DNS particulier
# request subnet-mask, routers, domain-name, domain-name-servers;  # liste des options souhaitées
```

## E. **/var/lib/dhcp/dhclient.<iface>.leases** — baux vus par le client

**Rôle :** historique local des baux obtenus (utile si le serveur refuse un nouveau bail car le client recycle un ancien).

---

# 3) Où lire les **logs**

* `journalctl -u isc-dhcp-server -n 50 --no-pager` (systemd/journald)
* (selon syslog) `sudo tail -f /var/log/syslog | grep -i dhcp`

> Tu y vois clairement **DORA** : `DHCPDISCOVER`, `DHCPOFFER`, `DHCPREQUEST`, `DHCPACK`.

---

## Mini pense-bête (quand ça ne démarre pas)

* **Fichier clé** : `/etc/dhcp/dhcpd.conf` → a-t-il bien un bloc `subnet` qui correspond **à l’IP de l’interface** ?
* **Interface** : `/etc/default/isc-dhcp-server` → `INTERFACESv4="enp0s8"` ?
* **État** : `systemctl status isc-dhcp-server` / `journalctl -u …`
* **Concurrence** : pas d’autre DHCP (VirtualBox) sur le même segment.

---

Si tu veux, je te fournis un **`dhcpd.conf` complet** et commenté pour **ton lab 10.10.10.0/24** (avec réservations vm2/vm3 et une variante avec option `routers` + NAT via vm1).
