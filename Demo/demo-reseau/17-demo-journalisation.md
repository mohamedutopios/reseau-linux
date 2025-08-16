Parfait 👍 tu veux maintenant ajouter la partie **Journalisation** dans ton lab, avec un focus sur les **logs réseau**.
C’est un complément idéal après la supervision et la sécurité SSH.

---

# 📜 1) Où trouver les logs réseau sous Linux (Ubuntu/Debian) ?

* **`/var/log/syslog`**
  → journal système global, contient beaucoup d’événements réseau (interfaces, DHCP client, routage, firewall si configuré pour logguer).

* **`/var/log/auth.log`**
  → journal d’authentification : connexions SSH réussies ou échouées, sudo, etc.
  Utile pour voir les attaques/brute force ou tester Fail2Ban.

* **`journalctl`** (si systemd est activé)
  → accès aux logs système (équivalent centralisé).
  Exemples :

  ```bash
  sudo journalctl -u ssh
  sudo journalctl -u networking
  ```

---

# 🔎 2) Exemple d’analyse de logs réseau

### 🔹 Connexions SSH (`/var/log/auth.log`)

Quand quelqu’un tente de se connecter :

```
Aug 16 10:21:43 vm2 sshd[1234]: Failed password for invalid user test from 10.10.10.11 port 52544 ssh2
Aug 16 10:21:46 vm2 sshd[1234]: Accepted publickey for vagrant from 10.10.10.11 port 52548 ssh2
```

👉 On peut extraire les tentatives échouées :

```bash
grep "Failed password" /var/log/auth.log
```

---

### 🔹 Logs réseau généraux (`/var/log/syslog`)

* Exemple : activation interface réseau :

```
Aug 16 09:12:31 vm1 systemd-networkd[452]: enp0s8: Gained carrier
Aug 16 09:12:32 vm1 dhclient[789]: DHCPACK from 10.10.10.12
```

* Exemple : `ufw` ou `iptables` avec LOG activé :

```
Aug 16 09:30:44 vm2 kernel: [12345.6789] [UFW BLOCK] IN=enp0s8 OUT= MAC=... SRC=10.10.10.11 DST=10.10.10.12 ...
```

👉 Recherche dans syslog :

```bash
grep "UFW" /var/log/syslog
grep "dhclient" /var/log/syslog
```

---

# 🛠️ 3) Exercices pratiques dans ton lab

### Exercice 1 : SSH réussi/échoué

* Depuis vm1 :

  ```bash
  ssh fakeuser@10.10.10.12
  ssh vagrant@10.10.10.12   # correct
  ```
* Sur vm2 :

  ```bash
  tail -f /var/log/auth.log
  ```

👉 Voir la différence entre “Failed password” et “Accepted publickey/password”.

---

### Exercice 2 : DHCP

* Sur vm1, renouvelle l’adresse IP :

  ```bash
  sudo dhclient -r enp0s8
  sudo dhclient enp0s8
  ```
* Sur vm1 :

  ```bash
  grep "DHCP" /var/log/syslog
  ```

👉 Observer la requête et la réponse DHCP.

---

### Exercice 3 : Pare-feu

* Sur vm2, active ufw :

  ```bash
  sudo ufw enable
  sudo ufw deny 22/tcp
  ```
* Depuis vm1, tente un SSH vers vm2.
* Sur vm2 :

  ```bash
  grep "UFW BLOCK" /var/log/syslog
  ```

👉 Voir l’entrée indiquant le blocage.

---

### Exercice 4 : Surveillance en temps réel

* Sur vm2 :

  ```bash
  tail -f /var/log/syslog /var/log/auth.log
  ```
* Depuis vm1, génère du trafic (ping, ssh, curl).
  👉 Les événements réseau apparaissent en direct.

---

# ✅ Résumé pédagogique

* **/var/log/auth.log** → suivi des connexions SSH (réussies, échouées).
* **/var/log/syslog** → événements réseau généraux (interfaces, DHCP, firewall).
* **journalctl** → alternative systemd moderne.
* Utile pour **sécurité** (intrusions), **diagnostic** (interfaces, DHCP) et **supervision** (trafic bloqué).

---

Veux-tu que je prépare un **TP complet “Journalisation réseau”** (énoncé + corrigé) où les apprenants doivent :

1. provoquer des erreurs SSH,
2. renouveler une IP via DHCP,
3. déclencher des blocages firewall,
4. analyser les logs correspondants en temps réel ?
