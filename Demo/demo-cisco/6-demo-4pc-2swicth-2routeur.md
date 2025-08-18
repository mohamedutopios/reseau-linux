Parfait 👌 je vais te donner une **démonstration complète dans Cisco Packet Tracer** avec :

* **4 PC**
* **2 Switchs**
* **2 Routeurs**
* Routage **statique** entre les réseaux.

---

# 🖥️ Topologie (vue logique)

```
 PC1 ---- SW1 ---- R1 ---- R2 ---- SW2 ---- PC3
 PC2 ---- SW1                       SW2 ---- PC4
```

---

# 📌 Étape 1 : Adressage IP (plan)

| Équipement | Interface       | Adresse IP   | Masque        |
| ---------- | --------------- | ------------ | ------------- |
| **PC1**    | NIC             | 192.168.1.10 | 255.255.255.0 |
| **PC2**    | NIC             | 192.168.1.11 | 255.255.255.0 |
| **PC3**    | NIC             | 192.168.3.10 | 255.255.255.0 |
| **PC4**    | NIC             | 192.168.3.11 | 255.255.255.0 |
| **R1**     | Fa0/0 (LAN1)    | 192.168.1.1  | 255.255.255.0 |
| **R1**     | Fa0/1 (vers R2) | 192.168.2.1  | 255.255.255.0 |
| **R2**     | Fa0/0 (vers R1) | 192.168.2.2  | 255.255.255.0 |
| **R2**     | Fa0/1 (LAN2)    | 192.168.3.1  | 255.255.255.0 |

---

# 📌 Étape 2 : Configuration des PC

* Dans **PC1** :

  * IP : `192.168.1.10`
  * Masque : `255.255.255.0`
  * Passerelle : `192.168.1.1`

* Dans **PC2** :

  * IP : `192.168.1.11`
  * Masque : `255.255.255.0`
  * Passerelle : `192.168.1.1`

* Dans **PC3** :

  * IP : `192.168.3.10`
  * Masque : `255.255.255.0`
  * Passerelle : `192.168.3.1`

* Dans **PC4** :

  * IP : `192.168.3.11`
  * Masque : `255.255.255.0`
  * Passerelle : `192.168.3.1`

---

# 📌 Étape 3 : Configuration des Routeurs

👉 Sur **R1**

```bash
enable
configure terminal

! Interface LAN (vers PC1 et PC2)
interface fastEthernet 0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit

! Interface vers R2
interface fastEthernet 0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
exit

! Route statique pour atteindre LAN2 (192.168.3.0/24 via R2)
ip route 192.168.3.0 255.255.255.0 192.168.2.2

end
write memory
```

👉 Sur **R2**

```bash
enable
configure terminal

! Interface vers R1
interface fastEthernet 0/0
 ip address 192.168.2.2 255.255.255.0
 no shutdown
exit

! Interface LAN (vers PC3 et PC4)
interface fastEthernet 0/1
 ip address 192.168.3.1 255.255.255.0
 no shutdown
exit

! Route statique pour atteindre LAN1 (192.168.1.0/24 via R1)
ip route 192.168.1.0 255.255.255.0 192.168.2.1

end
write memory
```

---

# 📌 Étape 4 : Vérification

1. Depuis **PC1** :

   ```
   ping 192.168.1.11   (PC2 → OK)
   ping 192.168.3.10   (PC3 → OK, via R1+R2)
   ping 192.168.3.11   (PC4 → OK)
   ```

2. Depuis **R1** :

   ```
   show ip route
   ```

   Tu dois voir :

   * `C 192.168.1.0/24` (connected LAN1)
   * `C 192.168.2.0/24` (link to R2)
   * `S 192.168.3.0/24 [1/0] via 192.168.2.2` (static route vers LAN2)

---

✅ Résultat : Tous les PC peuvent communiquer entre eux grâce au **routage statique**.

---

👉 Veux-tu que je te prépare un **schéma graphique Packet Tracer** avec câblage exact (ports Fa0/0, Fa0/1, etc.) pour que tu puisses le recréer directement dans ton projet ?
