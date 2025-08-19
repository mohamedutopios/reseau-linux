Parfait — on garde **ton lab 3 VMs** (Ubuntu 22.04) :

* **vm1** = `10.10.10.11` (client de test)
* **vm2** = `10.10.10.12` (serveur UDP pour la démo)
* **vm3** = `10.10.10.13` (serveur web Apache)

Je te donne pour **chaque étape** :

* le **pourquoi**,
* les **commandes**,
* **ce qu’on doit voir**,
* et **quoi faire si ça ne marche pas**.

---

# 0) Préparation (une seule fois par VM)

**Pourquoi ?** Avoir les outils modernes et classiques.

```bash
sudo apt-get update
sudo apt-get install -y iproute2 lsof net-tools netcat-openbsd curl
# iproute2 = ss, ip, etc. (déjà présent)
# lsof     = lsof
# net-tools= netstat
# netcat   = nc
# curl     = tester HTTP
```

---

# 1) Vérifier SSH (sur **toutes** les VMs)

### Pourquoi ?

SSH est déjà présent et **écoute sur 22**. Parfait pour comprendre les options `ss`/`netstat`/`lsof`.

### Commandes

```bash
# voir qui écoute (TCP/UDP/Listen/Process/Numeric)
sudo ss -tulpn | grep ':22'

# équivalent "ancien"
sudo netstat -tulpn | grep ':22'

# lister les fichiers réseaux du process qui écoute le 22
sudo lsof -iTCP:22 -sTCP:LISTEN -P -n
```

### Décryptage options (très important)

* `-t` TCP, `-u` UDP, `-l` LISTEN seulement, `-p` affiche le **PID/nom de process**, `-n` **pas de résolution DNS** (sorties rapides et claires), `-P` ne résout **pas** les ports en noms (lsof).
* Avec `ss`, colonnes utiles :
  `Netid` (tcp/udp), `State` (LISTEN), `Local Address:Port`, `Process` (pid/prog).

### Attendu

* Une ligne `LISTEN` sur `0.0.0.0:22` ou `*:22` (toutes interfaces) par `sshd`.

### Si KO

* `systemctl status ssh` (le service est-il démarré ?)
* `sudo systemctl start ssh`
* Port déjà pris ? → `sudo ss -tulpn | grep ':22'` (voir quel process)

---

# 2) Installer **Apache** sur **vm3** (HTTP/80)

### Pourquoi ?

Avoir un **service TCP réel** (HTTP) pour observer écoute + connexions.

### Commandes (vm3)

```bash
sudo apt-get install -y apache2

# écoute sur 80 ?
sudo ss -tulpn | grep ':80'
sudo lsof -iTCP:80 -sTCP:LISTEN -P -n

# tester localement
curl -I http://127.0.0.1/
```

### Depuis **vm1** (client)

```bash
curl -I http://10.10.10.13/
curl http://10.10.10.13/ | head
```

### Attendu

* `ss`/`lsof` : `apache2` (www-data) **LISTEN** sur `*:80`.
* `curl -I` : `HTTP/1.1 200 OK`.

### Si KO

* Service : `sudo systemctl status apache2` / `sudo journalctl -u apache2 -n 50`
* Port occupé : `sudo ss -tulpn | grep ':80'` (quel process ?)
* Écoute sur 127.0.0.1 seulement ?
  → vérifier `/etc/apache2/ports.conf` (Directive `Listen 80`) et VirtualHosts.

**Astuce “lecture fine” :**

```bash
# sockets TCP LISTEN avec info PID/UID
sudo ss -ltnp
# connexions établies vers Apache (après un curl depuis vm1)
sudo ss -tnp state established '( sport = :80 )'
```

---

# 3) Démarrer un **service UDP** (port 5000) sur **vm2**

### Pourquoi ?

Voir la différence **TCP vs UDP** (pas d’état “connected” côté UDP).

### Commandes (vm2)

```bash
# serveur UDP (lu = listen UDP)
nc -lu 5000
# (la commande reste bloquée en attente de datagrammes)
```

### Depuis **vm1** (client)

```bash
# envoi d’un datagramme UDP
echo "hello depuis vm1" | nc -u 10.10.10.12 5000

# sonder qu’un port UDP répond (nc OpenBSD)
nc -vzu 10.10.10.12 5000
```

### Observer l’écoute UDP (vm2)

```bash
sudo ss -lun | grep 5000           # écoute UDP
sudo lsof -iUDP:5000 -P -n         # process et socket
```

### Attendu

* Sur la console de **vm2** où `nc -lu 5000` tourne, tu vois “hello depuis vm1”.
* `ss -lun` montre `udp   UNCONN   0   0   *:5000   *:*` (UDP, pas de session établie).

### Si KO

* Mauvais paquet `nc` ? (`netcat-openbsd` OK)
* Firewall (rare sur Ubuntu par défaut) : `sudo ufw status`
* IP cible erronée ?

**Astuce :** UDP n’a **pas** d’état `ESTABLISHED`. Pour “voir” l’activité, lis la console `nc` serveur ou sniffe (voir bonus tcpdump plus bas).

---

# 4) Lire les **connexions actives** et qui parle à qui

### Pourquoi ?

Comprendre quelles **machines** sont connectées, sur **quels ports**.

### Commandes (ex. sur vm3 quand vm1 fait un curl)

```bash
# connexions TCP actives vers le port 80 local
sudo ss -tnp state established '( sport = :80 )'

# toutes les connexions établies TCP
sudo ss -tnp state established

# anciennes habitudes (si tu préfères)
sudo netstat -tnp
```

### Décryptage

* `state established` restreint aux connexions actives (flux TCP bidirectionnels).
* `sport = :80` = socket local **source** port 80 (donc serveur web).
* Ajoute `dport` pour filtrer sur le port distant :

  * `( dport = :80 )` → connexions **vers** le 80 distant (client HTTP).

---

# 5) Identifier **quel process** tient un port

### Pourquoi ?

Trouver “qui bloque un port”, ou “quel service écoute”.

### Commandes

```bash
# port 80 (Apache)
sudo lsof -iTCP:80 -sTCP:LISTEN -P -n
# port 22 (SSH)
sudo lsof -iTCP:22 -sTCP:LISTEN -P -n
# port 5000 UDP (netcat serveur)
sudo lsof -iUDP:5000 -P -n
```

### Attendu

* Tu vois `COMMAND`, `PID`, `USER`, `FD`, `TYPE`, `DEVICE`, `NODE`, `NAME` (avec l’IP\:PORT).

### Si tu dois **libérer un port**

```bash
sudo fuser -k 80/tcp     # tue le process qui tient 80 (si fuser installé)
# ou
sudo kill -9 <PID>
```

---

# 6) Bonus visibilité : sniffer vite

### Pourquoi ?

Voir le trafic **en direct** pour prouver ce qui circule.

```bash
# sur vm3 (serveur web), observe les requêtes HTTP plain-text locales
sudo tcpdump -i enp0s8 -n 'tcp port 80'
# sur vm2 (serveur UDP), observe les datagrammes UDP:5000
sudo tcpdump -i enp0s8 -n 'udp port 5000'
```

**Astuce pédagogique :** lance `tcpdump`, puis exécute `curl` (HTTP) ou `nc -u` (UDP) depuis vm1 → tu vois les paquets arriver (IP source/dest + ports).

---

# 7) Récap des **usages concrets** (le “pourquoi”)

* **`ss -tulpn`** : “Qui écoute ?” → Debug **démarrage de service**, **port occupé**, **mauvaise interface** (127.0.0.1 vs 0.0.0.0).
* **`lsof -i :PORT`** : “Quel **PID/process** tient le port ?” → Pour **arrêter**, **relancer**, ou **changer la conf**.
* **`ss state established`** : “Qui est **connecté maintenant** ?” → Vérifier qu’un client atteint bien le serveur.
* **`nc -lu` / `nc -u`** : **UDP** rapide → comprendre **sans connexion** (un seul datagramme suffit).
* **`curl`** : tester **HTTP** proprement (HEAD/GET, codes 200/301/404).
* **`tcpdump`** : preuve **indéniable** que le trafic passe (ou pas).

---

## Petites variantes utiles

* **Voir aussi l’UID/Utilisateur** :

  ```bash
  sudo ss -tulpen   # (eUid affiché)
  ```
* **Forcer IPv4** :

  ```bash
  sudo ss -tulpn 'inet'   # évite les sockets IPv6 si ça brouille la lecture
  ```
* **Lister tout, tri par port** :

  ```bash
  sudo ss -ltnp | sort -k4,4
  ```

---

# Ce que tu peux demander aux apprenants (mini-exos)

1. **Quel process** écoute sur 80 ? Sur 22 ?
   → `ss -tulpn`, `lsof -i :80`, `lsof -i :22`.

2. **Montre une connexion HTTP** en cours entre vm1 (client) et vm3 (serveur).
   → `curl` côté vm1, `ss -tnp state established '( sport = :80 )'` côté vm3.

3. **Prouve** qu’UDP5000 reçoit bien un datagramme.
   → `nc -lu 5000` sur vm2, `echo ... | nc -u 10.10.10.12 5000` sur vm1, `ss -lun` et/ou `tcpdump`.

4. **Le port 80 est occupé par un autre process** : identifie-le, tue-le, relance Apache.
   → `lsof -i :80`, `kill`, `systemctl restart apache2`.

---

Si tu veux, je te fournis un **énoncé TP “clé en main”** (PDF/markdown) avec : objectifs, pas-à-pas, captures attendues, et grille de correction.
