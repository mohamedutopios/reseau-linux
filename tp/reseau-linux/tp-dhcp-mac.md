Parfait 👌 on va se faire une **démo pratique** avec 2 VMs dans ton lab Vagrant (ou VirtualBox/KVM peu importe) :

* **vm1** : Client (Ubuntu)
* **vm2** : Serveur DHCP (Ubuntu)

Le but 👉 montrer que si on change l’adresse **MAC** du client, il recevra **une nouvelle IP** du serveur DHCP.

---

## 1️⃣ Préparation des VMs

### vm2 (Serveur DHCP)

1. Installer le serveur DHCP :

   ```bash
   sudo apt update
   sudo apt install isc-dhcp-server -y
   ```

2. Éditer la configuration `/etc/dhcp/dhcpd.conf` :

   ```bash
   default-lease-time 600;
   max-lease-time 7200;

   subnet 10.10.10.0 netmask 255.255.255.0 {
       range 10.10.10.100 10.10.10.200;
       option routers 10.10.10.1;
       option domain-name-servers 8.8.8.8;
   }
   ```

3. Spécifier l’interface utilisée (par ex. `enp0s8`) dans `/etc/default/isc-dhcp-server` :

   ```bash
   INTERFACESv4="enp0s8"
   ```

4. Redémarrer le service :

   ```bash
   sudo systemctl restart isc-dhcp-server
   sudo systemctl status isc-dhcp-server
   ```

---

### vm1 (Client DHCP)

1. Vérifier le nom de l’interface réseau (ex. `enp0s8`).

2. Lancer une requête DHCP :

   ```bash
   sudo dhclient -v enp0s8
   ip addr show enp0s8
   ```

   ➝ Le client devrait recevoir une IP (par ex. `10.10.10.100`).

3. Vérifier dans le serveur DHCP les baux attribués :

   ```bash
   sudo tail -f /var/lib/dhcp/dhcpd.leases
   ```

---

## 2️⃣ Démonstration : Changement de MAC

### Étape A — IP initiale

* Sur le client (`vm1`) :

  ```bash
  ip addr show enp0s8 | grep ether   # MAC actuelle
  ip addr show enp0s8 | grep inet    # IP actuelle (ex: 10.10.10.100)
  ```

### Étape B — Changer de MAC

1. Mettre l’interface down :

   ```bash
   sudo ip link set enp0s8 down
   ```
2. Changer l’adresse MAC :

   ```bash
   sudo ip link set enp0s8 address 02:11:22:33:44:55
   ```
3. Remettre l’interface up et demander une nouvelle IP :

   ```bash
   sudo ip link set enp0s8 up
   sudo dhclient -v enp0s8
   ```

### Étape C — Nouvelle IP

* Vérifier :

  ```bash
  ip addr show enp0s8 | grep inet
  ```

  ➝ Le client reçoit une **nouvelle IP** (par ex. `10.10.10.101`) car le serveur DHCP voit une nouvelle identité (nouvelle MAC).

* Sur le serveur DHCP :

  ```bash
  cat /var/lib/dhcp/dhcpd.leases
  ```

  ➝ On verra deux baux différents attribués au même client mais avec 2 adresses MAC distinctes.

---

## 3️⃣ Résultat attendu

* Avant changement : `10.10.10.100` attribuée à MAC `08:00:27:xx:xx:xx`
* Après changement : `10.10.10.101` attribuée à MAC `02:11:22:33:44:55`

✅ Preuve concrète que **changer de MAC = nouvelle identité réseau** pour un serveur DHCP.

---

👉 Veux-tu que je t’écrive un **Vagrantfile complet avec 2 VMs déjà configurées (client + serveur DHCP)** pour que tu puisses lancer la démo directement ?
