Parfait — voici, pour **chaque commande**, **ce qu’elle fait** (fonctionnement) et **à quoi elle sert** (cas d’usage), avec **mini-exemples** adaptés à ton lab (vm1=10.10.10.11, vm2=10.10.10.12, vm3=10.10.10.13).

---

## ip link

**Ce que fait :** parle au noyau (netlink) pour **lister/configurer les interfaces L2** (état, MTU, MAC, qdisc).
**À quoi ça sert :** diagnostiquer une carte, **simuler une panne** (down), tester MTU, **changer de MAC** (spoof).
**Exemples :**

* État et compteurs : `ip -s link show enp0s8`
* Couper/activer : `sudo ip link set enp0s8 down|up`
* MTU : `sudo ip link set enp0s8 mtu 1400`
* MAC : `sudo ip link set enp0s8 address 02:11:22:33:44:55`

---

## ip addr

**Ce que fait :** gère les **adresses IP** sur une interface (ajout, retrait, flush).
**À quoi ça sert :** passer une VM en IP fixe, **multi-IP**, remettre à zéro avant DHCP, basculer d’IP.
**Exemples :**

* Voir l’IP : `ip a s enp0s8`
* Multi-IP : `sudo ip a add 10.10.10.222/24 dev enp0s8`
* Changer totalement : `sudo ip a flush dev enp0s8; sudo ip a add 10.10.10.99/24 dev enp0s8`

---

## ip route

**Ce que fait :** lit/modifie la **table de routage** (next-hops, default).
**À quoi ça sert :** définir la **passerelle**, ajouter une **route statique**, simuler une panne L3.
**Exemples :**

* Table : `ip r`
* Default via vm2 : `sudo ip r replace default via 10.10.10.12`
* Route vers 192.168.50.0/24 via vm3 : `sudo ip r add 192.168.50.0/24 via 10.10.10.13`
* Panne : `sudo ip r del 10.10.10.0/24`

---

## ip neigh (ARP)

**Ce que fait :** affiche/vidange le **cache ARP** (IP↔MAC).
**À quoi ça sert :** vérifier la **résolution L2**, chasser une entrée pour retester.
**Exemples :**

* Voir : `ip neigh show`
* Forcer une nouvelle résolution : `sudo ip neigh flush all`

---

## ping

**Ce que fait :** envoie des **ICMP Echo** et mesure le **RTT**.
**À quoi ça sert :** tester la **connectivité L3**, la **latence/jitter**, la **MTU** (DF).
**Exemples :**

* LAN : `ping -c3 10.10.10.12`
* MTU : `ping -M do -s 1472 10.10.10.12`
* Intervalle : `ping -i 0.2 -c 10 10.10.10.13`

---

## traceroute

**Ce que fait :** découvre le **chemin** en jouant sur le **TTL** (UDP par défaut, ICMP avec `-I`, TCP avec `-T`).
**À quoi ça sert :** localiser où **ça bloque**, comparer **LAN (1 saut)** vs **WAN (multi-sauts)**, contourner filtrages.
**Exemples :**

* LAN : `traceroute -n 10.10.10.12`
* ICMP : `traceroute -I 10.10.10.12`
* TCP 443 : `traceroute -T -p 443 google.com`

---

## dig

**Ce que fait :** interroge le **DNS** (résolutions A/AAAA, MX, NS, reverse, trace).
**À quoi ça sert :** vérifier la **résolution**, différentier **DNS vs réseau**, inspecter enregistrements.
**Exemples :**

* Court : `dig +short openai.com`
* DNS précis : `dig @1.1.1.1 debian.org`
* Reverse : `dig -x 8.8.8.8 +short`

---

## ss (successeur de netstat)

**Ce que fait :** liste les **sockets** (LISTEN, ESTABLISHED) avec **PID/process**.
**À quoi ça sert :** “qui écoute ?”, “qui est connecté ?”, **quel service tient le port ?**
**Exemples :**

* Qui écoute : `sudo ss -tulpn`
* Connexions web vers vm3 : `sudo ss -tnp state established '( sport = :80 )'`

---

## lsof

**Ce que fait :** liste les **fichiers ouverts** par les processus (inclut les **sockets réseau**).
**À quoi ça sert :** mapper **port ↔ PID**, tuer le bon process, vérifier un bind sur 127.0.0.1 vs 0.0.0.0.
**Exemples :**

* Port 80 : `sudo lsof -iTCP:80 -sTCP:LISTEN -P -n`
* UDP 5000 : `sudo lsof -iUDP:5000 -P -n`

---

## netstat (legacy)

**Ce que fait :** ancien outil sockets/routes/interfaces.
**À quoi ça sert :** environ la même chose que `ss`, utile si habitudes/scripts.
**Exemples :**

* Écoute : `sudo netstat -tulpn`
* Routes : `netstat -rn`

---

## nc (netcat)

**Ce que fait :** ouvre des **connexions TCP/UDP** en mode client/serveur simple.
**À quoi ça sert :** tester **vite** un port/service, envoyer/recevoir un datagramme, **debug UDP**.
**Exemples :**

* Serveur UDP vm2 : `nc -lu 5000`
* Client UDP vm1 → vm2 : `echo hi | nc -u 10.10.10.12 5000`
* Test port TCP : `nc -vz 10.10.10.13 80`

---

## curl

**Ce que fait :** client **HTTP/HTTPS** (et autres), télécharge, affiche en-têtes.
**À quoi ça sert :** tester un **service web**, voir **codes HTTP**, suivre redirections, POSTer.
**Exemples :**

* En-têtes : `curl -I http://10.10.10.13/`
* Verbeux : `curl -v http://10.10.10.13/`

---

## tcpdump

**Ce que fait :** capture les **paquets** selon un **filtre BPF** (sniffer CLI).
**À quoi ça sert :** **prouver** qu’un trafic passe/ne passe pas, voir **DORA** DHCP, ARP, HTTP en clair.
**Exemples :**

* HTTP : `sudo tcpdump -i enp0s8 -nn 'tcp port 80'`
* DHCP : `sudo tcpdump -i enp0s8 -nn 'udp and (port 67 or 68)'`
* Vers PCAP : `-w cap.pcap` (analyse Wireshark)

---

## dhclient (client DHCP)

**Ce que fait :** demande/renouvelle/libère un **bail DHCP** pour une interface.
**À quoi ça sert :** forcer un **nouveau bail**, démo **DORA** côté client, debug DHCP.
**Exemples :**

* Libérer/puis demander : `sudo dhclient -r enp0s8 && sudo dhclient -v enp0s8`

---

## isc-dhcp-server (service)

**Ce que fait :** **serveur DHCP** (attribue IP + options), lit `dhcpd.conf`, tient `dhcpd.leases`.
**À quoi ça sert :** distribuer IPs au lab, **réservations MAC**, pousser DNS/gateway.
**Exemples :**

* Lancer : `sudo systemctl restart isc-dhcp-server`
* Logs : `journalctl -u isc-dhcp-server -n 50`
* Leases : `sudo tail -n +1 /var/lib/dhcp/dhcpd.leases`

---

## iptables

**Ce que fait :** programme les **règles de pare-feu/NAT** (netfilter).
**À quoi ça sert :** **bloquer/autoriser** un trafic, faire du **NAT** (MASQUERADE) depuis le lab.
**Exemples :**

* Bloquer ICMP in : `sudo iptables -I INPUT -p icmp -j DROP`
* NAT via enp0s3 : `sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE`

---

## netplan

**Ce que fait :** lit des **YAML** dans `/etc/netplan/` et **génère** la conf pour **systemd-networkd** ou **NetworkManager**.
**À quoi ça sert :** activer **DHCP**, définir **IP statique**, routes, DNS, VLAN, de façon **déclarative**.
**Exemples :**

* DHCP sur enp0s8 :

  ```yaml
  network:
    version: 2
    ethernets:
      enp0s8: { dhcp4: true }
  ```

  Appliquer : `sudo netplan apply`
* Sécurisé à distance : `sudo netplan try`

---

### Petit pense-bête “choisir l’outil”

* **Carte OK ?** → `ip link`
* **Adresse OK ?** → `ip addr`
* **Route OK ?** → `ip route`
* **L2 résolu ?** → `ip neigh`
* **Ping OK ?** → réseau L3 OK
* **Chemin ?** → `traceroute`
* **DNS ?** → `dig`
* **Qui écoute/qui parle ?** → `ss` / `lsof`
* **Tester service** → `curl` / `nc`
* **Prouver paquet** → `tcpdump`
* **DHCP client/serveur** → `dhclient` / `isc-dhcp-server`
* **Firewall/NAT** → `iptables`
* **(Re)config réseau** → `netplan`

Si tu veux, je te mets tout ça en **fiche A4** (PDF) prête à distribuer aux apprenants.
