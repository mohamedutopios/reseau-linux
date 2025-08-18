Parfait 🚀 tu veux faire une **démo Packet Tracer avec `tracert` (traceroute)**.
Je vais te donner **toutes les étapes précises** : câblage, configuration, test, et ce que tu dois voir.

---

# 🖥️ Topologie demandée

* **PC1** ↔ **Switch1** ↔ **Routeur** ↔ **Switch2** ↔ **PC2**
* Adresse IP :

  * PC1 : `192.168.1.10 /24`, GW `192.168.1.1`
  * Interface routeur côté PC1 : `192.168.1.1 /24`
  * Interface routeur côté PC2 : `192.168.2.1 /24`
  * PC2 : `192.168.2.10 /24`, GW `192.168.2.1`

---

# 🔧 Étapes détaillées

## 1️⃣ Créer la topologie dans Packet Tracer

1. Dans **End Devices**, ajoute 2 **PC** (PC1 et PC2).
2. Dans **Switches**, ajoute 2 **2960** (Switch1 et Switch2).
3. Dans **Routers**, prends un **Router 2911** (par exemple).
4. Relie :

   * PC1 ↔ Switch1 avec câble droit (`Copper Straight-Through`)
   * Switch1 ↔ Routeur (FastEthernet 0/0)
   * Routeur (FastEthernet 0/1) ↔ Switch2
   * Switch2 ↔ PC2 avec câble droit

---

## 2️⃣ Configurer les PC

👉 Clique sur chaque PC → **Desktop → IP Configuration**

* **PC1**

  ```
  IP Address : 192.168.1.10
  Subnet Mask: 255.255.255.0
  Default Gateway: 192.168.1.1
  ```

* **PC2**

  ```
  IP Address : 192.168.2.10
  Subnet Mask: 255.255.255.0
  Default Gateway: 192.168.2.1
  ```

---

## 3️⃣ Configurer le routeur

👉 Clique sur le routeur → **CLI**

### a) Activer les interfaces

```
enable
configure terminal
!
interface fastEthernet 0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit
!
interface fastEthernet 0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
exit
!
end
write memory
```

💡 Les LED passent du rouge à l’orange puis au vert = interfaces actives.

---

## 4️⃣ Vérification avec `ping`

* Depuis **PC1** → Ouvre **Command Prompt** et tape :

  ```
  ping 192.168.1.1    (test passerelle)
  ping 192.168.2.1    (test autre réseau via routeur)
  ping 192.168.2.10   (test PC2)
  ```

👉 Tous doivent répondre si la config est bonne.

---

## 5️⃣ Démo `tracert`

Toujours depuis **PC1** → Command Prompt :

```
tracert 192.168.2.10
```

### Résultat attendu

* 1er saut : `192.168.1.1` (le routeur, interface côté PC1)
* 2ème saut : `192.168.2.10` (PC2)

👉 Tu verras donc le chemin complet que le paquet prend.

---

# 🎯 Ce que tu peux montrer en démo

1. Les **trames ARP** au début (pour résoudre les adresses MAC).
2. Les **pings** qui prouvent la connectivité.
3. Le `tracert` qui affiche bien 2 sauts : passerelle → destination.
4. Eventuellement, dans **Simulation Mode**, observer chaque paquet ICMP Echo/Reply ou ICMP TTL Expired (utilisé par `tracert`).

---

Veux-tu que je te prépare aussi la **version en mode Simulation dans Packet Tracer** pour que tu voies étape par étape comment `tracert` envoie des paquets avec TTL croissant (1, 2, 3…) ?
