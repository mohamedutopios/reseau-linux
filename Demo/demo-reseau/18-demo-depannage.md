Parfait 👍 tu veux finir ton module avec une section **Dépannage** : mettre les apprenants dans des cas concrets de panne réseau (connectivité, DNS, passerelle, firewall) et leur faire appliquer les outils vus (`ping`, `ip addr`, `ip route`, `dig`, `ss`, `iptables`, logs…).

---

# 🛠️ 1) Méthodologie de dépannage réseau

1. **Vérifier la couche 1/2** : câble, interface UP ?

   ```bash
   ip link show enp0s8
   ```
2. **Vérifier l’adresse IP** :

   ```bash
   ip addr show enp0s8
   ```
3. **Vérifier la passerelle** :

   ```bash
   ip route
   ```
4. **Tester la connectivité IP (ICMP)** :

   ```bash
   ping 10.10.10.12
   ping 1.1.1.1
   ```
5. **Vérifier le DNS** :

   ```bash
   dig www.google.com
   cat /etc/resolv.conf
   ```
6. **Vérifier les services** :

   ```bash
   ss -tulpn
   ```
7. **Vérifier le firewall** :

   ```bash
   sudo iptables -L -n -v
   sudo ufw status verbose
   ```
8. **Analyser les logs** :

   ```bash
   tail -f /var/log/syslog /var/log/auth.log
   ```

---

# 🚧 2) Cas pratiques dans ton lab (vm1, vm2, vm3)

## 🔹 Cas 1 : Panne d’interface réseau

* **Simulation (vm2)** :

  ```bash
  sudo ip link set enp0s8 down
  ```
* **Symptôme (vm1)** : plus de ping vers 10.10.10.12.
* **Diagnostic attendu** : `ip link` → interface DOWN sur vm2.
* **Correction** :

  ```bash
  sudo ip link set enp0s8 up
  ```

---

## 🔹 Cas 2 : Mauvaise adresse IP

* **Simulation (vm3)** :

  ```bash
  sudo ip addr flush dev enp0s8
  sudo ip addr add 10.10.20.13/24 dev enp0s8
  ```
* **Symptôme (vm1)** : `ping 10.10.10.13` KO.
* **Diagnostic attendu** : `ip addr` → mauvaise IP.
* **Correction** :

  ```bash
  sudo ip addr flush dev enp0s8
  sudo ip addr add 10.10.10.13/24 dev enp0s8
  ```

---

## 🔹 Cas 3 : Passerelle absente

* **Simulation (vm1)** :

  ```bash
  sudo ip route del default
  ```
* **Symptôme** : ping LAN OK, ping Internet KO.
* **Diagnostic attendu** : `ip route` → pas de default route.
* **Correction** :

  ```bash
  sudo ip route add default via 10.10.10.12 dev enp0s8
  ```

---

## 🔹 Cas 4 : Problème DNS

* **Simulation (vm1)** : modifier `/etc/resolv.conf` :

  ```
  nameserver 10.10.10.254
  ```
* **Symptôme** : `ping 1.1.1.1` OK mais `ping www.google.com` KO.
* **Diagnostic attendu** : `dig www.google.com` → pas de réponse DNS.
* **Correction** :

  ```
  nameserver 1.1.1.1
  nameserver 8.8.8.8
  ```

---

## 🔹 Cas 5 : Firewall bloquant SSH

* **Simulation (vm2)** :

  ```bash
  sudo ufw enable
  sudo ufw deny 22/tcp
  ```
* **Symptôme (vm1)** : impossible de se connecter en SSH à vm2.
* **Diagnostic attendu** : `nmap -p 22 10.10.10.12` → port fermé/filtré.
* **Correction** :

  ```bash
  sudo ufw allow 22/tcp
  ```

---

# 🧪 3) Scénario TP "Dépannage"

### Objectif :

Les apprenants doivent identifier et corriger des pannes simulées sur les VMs en utilisant les outils vus.

### Exemple d’énoncé :

1. Depuis **vm1**, vous n’arrivez plus à joindre vm3. Trouvez et corrigez la panne.
2. Depuis **vm1**, vous accédez à Internet par IP (`ping 1.1.1.1`) mais pas par nom. Analysez.
3. Depuis **vm1**, le SSH vers vm2 échoue. Vérifiez le firewall.

---

# ✅ Résumé pédagogique

* **Couche 2** : interface DOWN, IP manquante.
* **Couche 3** : mauvaise route/passerelle.
* **Couche 4/7** : port bloqué par firewall.
* **DNS** : résolveur incorrect.
* **Logs** : toujours à vérifier pour confirmation.

---

Veux-tu que je t’écrive un **TP complet "Dépannage réseau"** (énoncé + corrigé) avec 4–5 scénarios progressifs, que tu pourras donner directement à tes apprenants ?
