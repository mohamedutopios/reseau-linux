Parfait — voilà un **mémo d’options “qui servent vraiment”** pour tes démos réseau, organisé **par commande** et adapté à ton lab 3 VMs (vm1=10.10.10.11, vm2=10.10.10.12, vm3=10.10.10.13).
Je mets pour chaque outil : **options clés → ce que ça fait → mini-exemple dans le lab**.

---

# ip (link / addr / route / neigh)

## Options communes utiles

* `-4` / `-6` : forcer IPv4/IPv6 (ex. `ip -4 a`).
* `-s` : stats détaillées (ex. `ip -s link show enp0s8`).
* `-br` : sortie “brief” lisible (ex. `ip -br a`).
* `-d` : infos “debug” (ex. `ip -d link show`).
* `-j` : JSON (pour scripts).

## ip link

* `show [dev IF]` : état/flags/MTU/MAC.
  ex. `ip -s link show enp0s8`
* `set dev IF up|down` : activer/couper.
  ex. `ip link set enp0s8 down`
* `set dev IF mtu N` : MTU.
  ex. `ip link set enp0s8 mtu 1400`
* `set dev IF address MAC` : changer la MAC.
  ex. `ip link set enp0s8 address 02:11:22:33:44:55`

## ip addr

* `show [dev IF]` : lister les IP.
  ex. `ip a s enp0s8`
* `add A.B.C.D/len dev IF [label IF:alias]` : ajouter.
  ex. `ip a add 10.10.10.222/24 dev enp0s8`
* `del A.B.C.D/len dev IF` : retirer.
  ex. `ip a del 10.10.10.222/24 dev enp0s8`
* `flush dev IF` : vider toutes les IP.
  ex. `ip a flush dev enp0s8`

## ip route

* `show` : table de routage.
  ex. `ip r`
* `add NET via GW [metric M]` : route statique.
  ex. `ip r add 192.168.50.0/24 via 10.10.10.13`
* `replace default via GW` : changer la passerelle.
  ex. `ip r replace default via 10.10.10.12`
* `del NET|default` : supprimer.
  ex. `ip r del default`

## ip neigh (ARP)

* `show` : cache ARP.
  ex. `ip neigh show`
* `flush all` : vider pour forcer une résolution.
  ex. `ip neigh flush all`

---

# ping

**Options clés**

* `-c N` : nombre de paquets. `ping -c3 10.10.10.12`
* `-i S` : intervalle (s). `ping -i 0.2 -c 10 10.10.10.12`
* `-s BYTES` : taille payload. `ping -s 1472 -M do 10.10.10.12`
* `-M do` : “don’t fragment” (tester MTU).
* `-W S` : timeout de réponse (s).
* `-I IF|ADDR` : interface/source. `ping -I enp0s8 10.10.10.12`
* `-4` / `-6` : forcer la famille.
* `-n` : pas de DNS (affiche IPs).

**Exemples**

* MTU : `ping -M do -s 1472 10.10.10.12` (OK si MTU=1500).
* Surveillance : `ping 10.10.10.11` (Ctrl-C pour stop).

---

# traceroute

**Modes**

* (défaut) **UDP** ports 33434+.
* `-I` : **ICMP Echo** (comme ping).
* `-T` : **TCP SYN**, `-p PORT` pour choisir (utile si ICMP/UDP filtrés).

**Autres options**

* `-n` : pas de DNS → + rapide.
* `-q N` : N sondes/saut (def. 3).
* `-w S` : timeout par sonde (s).
* `-m H` : TTL max (nb de sauts).
* `-f X` : TTL de départ (sauter les N premiers routeurs).
* `-4` / `-6` : famille IP.

**Exemples**

* LAN : `traceroute -n 10.10.10.12` (1 saut).
* TCP 443 : `traceroute -T -p 443 google.com`.
* ICMP strict : `traceroute -I 10.10.10.12`.

---

# dig (DNS)

**Forme générale**

* `dig NAME [TYPE]` (TYPE par défaut = A).

**Options & usages**

* `@DNS` : interroger un résolveur précis.
  `dig @1.1.1.1 openai.com`
* `+short` : sortie courte.
  `dig +short google.com`
* `-x IP` : reverse lookup.
  `dig -x 8.8.8.8 +short`
* `+trace` : itératif depuis la racine (diagnostic).
* `+tcp` : forcer TCP.
* `+time=SEC` / `+tries=N` : patience.

---

# ss (successeur de netstat)

**Affichages courants**

* `ss -tulpn` : **qui écoute** TCP/UDP + PID (serveurs).
* `ss -ltnp` : TCP listen only.
* `ss -lun` : UDP listen only.
* `ss -tnp state established` : connexions TCP actives.

**Filtres utiles**

* `'( sport = :80 )'` : port local 80 (serveur web).
* `'( dport = :80 )'` : port distant 80 (client http).
* `dst 10.10.10.11` / `src 10.10.10.12` : IPs.

**Options**

* `-t` TCP, `-u` UDP, `-l` LISTEN, `-a` tout, `-n` pas de DNS, `-p` PID/nom process, `-4/-6` famille, `-e` plus d’infos.

---

# lsof (ports ↔ processus)

**Usages**

* `lsof -iTCP:80 -sTCP:LISTEN -P -n` : qui tient 80 (écoute TCP).
* `lsof -iUDP:5000 -P -n` : socket UDP:5000.
* `lsof -p PID` : fichiers réseau ouverts par un PID.
* `-P` : ports en chiffres, `-n` : pas de DNS.

---

# netstat (ancien, encore pratique)

* `netstat -tulpn` : qui écoute (comme ss).
* `netstat -tnp` : connexions TCP actives.
* `netstat -rn` : table de routage.
* `netstat -i` : stats interfaces.

---

# nc (netcat-openbsd)

**Côtés**

* Serveur UDP : `nc -lu 5000`
* Serveur TCP : `nc -l 5000` (ajoute `-k` pour rester ouvert)
* Client UDP : `echo "hi" | nc -u 10.10.10.12 5000`
* Client TCP : `nc 10.10.10.13 80`

**Options clés**

* `-l` listen, `-k` rester ouvert après un client,
* `-u` UDP (sinon TCP), `-v` verbeux,
* `-z` scan “sans envoyer de données” (TCP),
* `-n` pas de DNS, `-p` port source,
* `-w S` timeout (s), `-q S` délais fin de flux.

**Exemples**

* Test port TCP ouvert : `nc -vz 10.10.10.13 80`
* Sonde UDP : `nc -vzu 10.10.10.12 5000`

---

# curl (HTTP/HTTPS)

**Options fréquentes**

* `-I` (HEAD) : en-têtes uniquement. `curl -I http://10.10.10.13/`
* `-v` : verbeux (handshake, headers).
* `-sS` : silencieux mais garde erreurs.
* `-L` : suivre redirections.
* `-o file` / `-O` : écrire sortie.
* `-k` : ignorer certifs (tests HTTPS).
* `-H 'Header: value'` : ajouter en-tête.
* `-d 'x=1&y=2'` / `--data-binary @file` : POST.
* `-X METHOD` : méthode explicite.
* `--resolve host:port:IP` : forcer DNS.
* `--interface enp0s8` : sortir par une interface.

**Exemples**

* Test dispo : `curl -I http://10.10.10.13/`
* Latence simple : `time curl -I http://10.10.10.13/`

---

# tcpdump

**Options essentielles**

* `-i IF` : interface (ex. `-i enp0s8`)
* `-n` : pas de DNS, `-nn` : pas de noms de ports non plus
* `-v/-vv` : verbeux (plus de champs)
* `-s 0` : snaplen max (ne tronque pas)
* `-c N` : nombre de paquets puis stop
* `-w file.pcap` : écrire un PCAP (pour Wireshark)
* `-r file.pcap` : lire un PCAP

**Filtres BPF utiles**

* `host 10.10.10.12` ; `src host …` ; `dst host …`
* `port 80` ; `tcp port 80` ; `udp port 5000`
* `icmp` ; `arp` ; `(port 67 or port 68)` (DHCP)

**Exemples**

* HTTP : `tcpdump -i enp0s8 -nn 'tcp port 80'`
* DHCP : `tcpdump -i enp0s8 -nn 'udp and (port 67 or 68)'`

---

# dhclient (côté client DHCP)

**Options utiles**

* `-r IF` : libère le bail. `dhclient -r enp0s8`
* `-v` : verbeux (détails DORA).
* `-4` / `-6` : IPv4/IPv6.
* `-1` : essaie une fois puis sort (utile en scripts).
* `IFACE` en dernier : interface ciblée.
  ex. `dhclient -v enp0s8`

---

# isc-dhcp-server (côté serveur)

**Fichiers & commandes**

* `/etc/dhcp/dhcpd.conf` : config (subnet, range, options, réservations).
* `/etc/default/isc-dhcp-server` : `INTERFACESv4="enp0s8"`.
* `systemctl [status|restart] isc-dhcp-server`
* `journalctl -u isc-dhcp-server -f` : logs live.
* Leases : `/var/lib/dhcp/dhcpd.leases`

---

# iptables (rappel de base)

**Tables & actions**

* `-t nat|filter|mangle` : choisir la table.
* `-A` ajouter, `-I` insérer (en tête), `-D` supprimer, `-R` remplacer.
* `-L -n -v` : lister règles (nombre de hits).
* `-C` : “check” (existe ?).

**Match & cible**

* `-p tcp|udp|icmp`, `--dport N`, `--sport N`
* `-s 10.10.10.0/24` / `-d …`
* `-j ACCEPT|DROP|REJECT|MASQUERADE`

**Exemples**

* Bloquer ICMP in : `iptables -I INPUT -p icmp -j DROP`
* NAT sortie : `iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE`

---

# netplan

**Commandes**

* `netplan get` : voir config fusionnée.
* `sudo netplan apply` : appliquer.
* `sudo netplan try` : applique avec rollback auto si non validé (sécurisé en SSH).
* `sudo netplan generate` : (re)génère pour le backend.
* `sudo netplan apply --debug` : débogage.

**YAML rapide**

```yaml
network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: true     # client DHCP
```

---

## Mini “antisèche” (3 lignes par besoin)

* **Qui écoute ?** `ss -tulpn` → PID/process ↔ port.
* **Qui est connecté ?** `ss -tnp state established` → flux TCP actifs.
* **Quel process ?** `lsof -i :PORT -P -n` → PID/USER précis.
* **HTTP OK ?** `curl -I http://10.10.10.13/` → 200/301/…
* **UDP arrive ?** `nc -lu 5000` (vm2) ⇄ `echo hi | nc -u vm2 5000` (vm1).
* **Voir paquet** `tcpdump -i enp0s8 -nn 'tcp port 80'`.

Si tu veux, je peux te transformer ça en **PDF “cheat-sheet”** propre à imprimer pour la formation.
