Super 👍 tu as totalement raison : j’ai montré les commandes, mais pas assez détaillé **pourquoi** ni **à quoi sert chaque option**.
On va reprendre proprement, avec **3 colonnes** :

👉 **Commande**, 👉 **Options détaillées**, 👉 **Intérêt concret (cas d’usage)**.

---

# 🔑 1) `ss` (successeur moderne de netstat)

```bash
sudo ss -tulpn
```

### Options

* `-t` = sockets TCP
* `-u` = sockets UDP
* `-l` = en **LISTEN** (en attente de connexion)
* `-p` = affiche le PID + nom du process
* `-n` = ne résout pas les noms de domaine/ports → affiche directement `22`, pas `ssh`

### Intérêt

* Savoir **quels services écoutent** (ex. Apache sur 80, SSH sur 22)
* Identifier un **port bloqué ou déjà utilisé**
* Vérifier si un service écoute sur **0.0.0.0** (toutes interfaces) ou seulement sur `127.0.0.1`

---

# 🔑 2) `netstat` (ancien mais encore utile)

```bash
sudo netstat -tulpn
```

### Options

* `-t` = TCP
* `-u` = UDP
* `-l` = LISTEN uniquement
* `-p` = PID/process
* `-n` = pas de résolution DNS

### Intérêt

* Même usage que `ss`, mais moins lisible.
* Pratique si `ss` n’est pas installé.
* Donne une vision classique et connue par beaucoup d’admins.

---

# 🔑 3) `lsof` (list open files)

```bash
sudo lsof -iTCP:80 -sTCP:LISTEN -P -n
```

### Options

* `-iTCP:80` = filtre sur sockets TCP port 80
* `-sTCP:LISTEN` = affiche seulement celles en écoute
* `-P` = ne convertit pas le port en nom (`80` au lieu de `http`)
* `-n` = ne fait pas de lookup DNS
* Sans `-i`, `lsof` montre **tous les fichiers ouverts**, pas seulement les sockets

### Intérêt

* Trouver **quel process exact** tient un port
* Exemple : si Apache refuse de démarrer → vérifier si **un autre process occupe le 80**
* Identifier **l’utilisateur** qui exécute le service (`www-data`, `root`, etc.)

---

# 🔑 4) `nc` (netcat)

### Serveur UDP

```bash
nc -lu 5000
```

* `-l` = mode serveur (listen)
* `-u` = UDP
* (par défaut TCP si pas `-u`)

👉 Sert à :

* Créer un petit serveur UDP/TCP de test.
* Vérifier que des paquets arrivent bien.

---

### Client UDP

```bash
echo "hello" | nc -u 10.10.10.12 5000
```

* `-u` = protocole UDP
* Sans `-u`, on enverrait en TCP.

👉 Sert à :

* Envoyer des messages de test vers un serveur.
* Simuler une appli cliente (exemple : un capteur IoT envoie en UDP).

---

### Scanner UDP

```bash
nc -vzu 10.10.10.12 5000
```

* `-v` = verbose (détaille le résultat)
* `-z` = mode “scan” (n’envoie rien de lisible, juste une sonde)
* `-u` = UDP

👉 Sert à :

* Tester si un port UDP est ouvert/répond.
* Alternative simplifiée à `nmap` pour un port précis.

---

# 🔑 5) `curl`

### Tester HTTP

```bash
curl -I http://10.10.10.13/
```

* `-I` = HEAD request (seulement les en-têtes HTTP, pas le contenu)

👉 Sert à :

* Vérifier si un site répond bien et avec quel **code HTTP** (200, 404, 500).
* Diagnostic rapide sans navigateur.

### Télécharger la page

```bash
curl http://10.10.10.13/ | head
```

👉 Sert à :

* Vérifier le contenu HTML rendu.
* Très utilisé dans les scripts d’automatisation.

---

# 🔑 6) `tcpdump`

```bash
sudo tcpdump -i enp0s8 -n 'tcp port 80'
```

### Options

* `-i enp0s8` = interface réseau à écouter
* `-n` = ne fait pas de résolution DNS (plus clair et rapide)
* `'tcp port 80'` = filtre BPF : ne capture que TCP port 80

👉 Sert à :

* Voir **en direct** si des paquets arrivent (preuve ultime du trafic).
* Diagnostiquer si la requête sort, mais ne revient pas.
* Différencier **TCP (handshake, connexions)** de **UDP (juste envoi brut)**.

---

# 🔑 7) `fuser`

```bash
sudo fuser -k 80/tcp
```

### Options

* `80/tcp` = cible le port 80 en TCP
* `-k` = tue le process qui occupe ce port

👉 Sert à :

* Libérer rapidement un port bloqué par un service zombie.
* Utile si Apache refuse de démarrer parce que “port 80 déjà utilisé”.

---

# 🎯 Résumé pédagogique

| Commande           | Sert à quoi ?                           | Cas concret                              |
| ------------------ | --------------------------------------- | ---------------------------------------- |
| `ss -tulpn`        | Voir qui écoute                         | Vérifier que SSH ou Apache écoutent bien |
| `netstat -tulpn`   | Ancien équivalent                       | Même usage que `ss`, pour compatibilité  |
| `lsof -i :PORT`    | Trouver le PID du process lié à un port | Identifier qui occupe 80                 |
| `nc -lu` / `nc -u` | Créer/tester un service UDP/TCP simple  | Vérifier un datagramme UDP               |
| `curl -I`          | Tester un service web                   | Vérifier un code HTTP 200                |
| `tcpdump`          | Sniffer le trafic                       | Prouver que la requête HTTP/UDP circule  |
| `fuser -k`         | Libérer un port                         | Tuer le process qui bloque 80            |

---

👉 Est-ce que tu veux que je refasse ton **TP complet** mais cette fois avec :

* tableau récap de toutes les commandes,
* explications détaillées des options,
* et **mises en situation concrètes** (comme : “Apache ne démarre pas, trouve pourquoi”) ?
