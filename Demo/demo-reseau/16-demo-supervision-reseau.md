Parfait 👍 tu veux compléter ton lab avec la partie **Supervision réseau**, pour observer en temps réel le trafic et analyser plus en détail avec `tcpdump`/Wireshark.
Voici une présentation claire + des exercices concrets adaptés à tes VMs (vm1/vm2/vm3).

---

# 📊 1) Supervision en temps réel du trafic

## 🔹 `iftop`

* **But** : voir le trafic réseau en temps réel (équivalent de `top` mais pour le réseau).
* Installation :

  ```bash
  sudo apt install -y iftop
  ```
* Utilisation :

  ```bash
  sudo iftop -i enp0s8
  ```

  👉 Affiche les flux **src → dst**, avec débits instantanés.

---

## 🔹 `nload`

* **But** : graphique en temps réel du débit entrant/sortant.
* Installation :

  ```bash
  sudo apt install -y nload
  ```
* Utilisation :

  ```bash
  sudo nload enp0s8
  ```

  👉 Simple et visuel (RX/TX en Kbit/s, Mbit/s).

---

## 🔹 `vnstat`

* **But** : statistiques cumulées (historique trafic par heure/jour/mois).
* Installation :

  ```bash
  sudo apt install -y vnstat
  sudo systemctl enable vnstat --now
  ```
* Initialiser interface :

  ```bash
  sudo vnstat -u -i enp0s8
  ```
* Utilisation :

  ```bash
  vnstat        # stats globales
  vnstat -d     # stats journalières
  vnstat -h     # stats horaires
  vnstat -l     # suivi en direct
  ```

---

# 🔎 2) Analyse détaillée du trafic

## 🔹 `tcpdump`

* **But** : capturer les paquets sur une interface.
* Installation :

  ```bash
  sudo apt install -y tcpdump
  ```
* Exemples :

  * Capturer 10 paquets ICMP :

    ```bash
    sudo tcpdump -i enp0s8 icmp -c 10
    ```
  * Capturer HTTP (port 80) :

    ```bash
    sudo tcpdump -i enp0s8 port 80 -c 10
    ```
  * Sauvegarder pour Wireshark :

    ```bash
    sudo tcpdump -i enp0s8 -w capture.pcap
    ```

## 🔹 Wireshark

* **But** : analyse graphique et détaillée des paquets (PCAP).
* Sur une VM graphique :

  ```bash
  sudo apt install -y wireshark
  ```
* Ou bien :

  * Capturer sur vm2/vm3 avec `tcpdump -w capture.pcap`.
  * Copier le fichier `.pcap` sur ta machine hôte.
  * L’ouvrir avec Wireshark → filtrer avec `icmp`, `http`, `ssh`…

---

# 🧪 3) Exercices pratiques dans ton lab

### Exercice 1 : Générer du trafic ICMP

* Depuis vm1 :

  ```bash
  ping -c 5 10.10.10.13
  ```
* Sur vm3 :

  ```bash
  sudo iftop -i enp0s8
  sudo tcpdump -i enp0s8 icmp
  ```

👉 Observer les paquets ping et réponses.

---

### Exercice 2 : Générer du trafic HTTP

* Installer Apache sur vm3 :

  ```bash
  sudo apt install -y apache2
  ```
* Depuis vm1 :

  ```bash
  curl http://10.10.10.13
  ```
* Sur vm3 :

  ```bash
  nload enp0s8
  sudo tcpdump -i enp0s8 port 80 -c 10
  ```

👉 Voir le trafic HTTP apparaître.

---

### Exercice 3 : Statistiques avec vnstat

* Sur vm2 :

  ```bash
  sudo vnstat -h
  ```

👉 Observer les stats horaires après avoir généré du trafic (ex. téléchargements depuis vm1).

---

### Exercice 4 (bonus) : Analyse avancée avec Wireshark

* Sur vm2 :

  ```bash
  sudo tcpdump -i enp0s8 -w traffic.pcap
  ```
* Depuis vm1 → lancer du trafic vers vm3 (ping, curl, ssh).
* Copier `traffic.pcap` sur ton PC hôte et analyser dans Wireshark.
  👉 Filtrer :

  * `icmp` → ping.
  * `http` → requêtes Apache.
  * `ssh` → handshake chiffré.

---

# ✅ Résumé pédagogique

* **iftop/nload** = supervision en temps réel (flux et débits).
* **vnstat** = suivi sur la durée (historique trafic).
* **tcpdump** = capture bas niveau (paquets).
* **Wireshark** = analyse graphique approfondie.

---

Veux-tu que je prépare un **TP complet “Supervision réseau”** (énoncé + corrigé), où l’apprenant doit :

1. Surveiller un ping avec `iftop`, `nload`, `tcpdump`.
2. Observer un accès web (HTTP).
3. Consulter des stats réseau avec `vnstat`.
4. Exporter et analyser un `.pcap` dans Wireshark ?
