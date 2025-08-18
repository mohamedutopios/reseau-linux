Super 👍 on passe au **cas concret du trunk** → c’est là que tu vois comment **plusieurs VLANs traversent un même câble entre deux switches**.

---

# 🔹 1. Objectif de la démo

* Faire **2 VLANs (10 et 20)**.
* Les étendre sur **2 switches reliés par un trunk**.
* Vérifier que :

  * PC1 (VLAN 10 sur SW1) peut parler avec PC3 (VLAN 10 sur SW2).
  * PC2 (VLAN 20 sur SW1) peut parler avec PC4 (VLAN 20 sur SW2).
  * Mais PC1 ne peut pas parler avec PC2/PC4 (pas le même VLAN).

---

# 🔹 2. Topologie

```
        +---------+              +---------+
PC1 ----| Fa0/1   |              |   Fa0/1 |---- PC3
        |         | Fa0/24---24  |         |
PC2 ----| Fa0/2   |              |   Fa0/2 |---- PC4
        +---------+              +---------+
           SW1                      SW2
```

* **VLAN 10 :** PC1 ↔ PC3
* **VLAN 20 :** PC2 ↔ PC4
* **Lien SW1 ↔ SW2 (Fa0/24)** → en mode **trunk**

---

# 🔹 3. Adressage IP

* PC1 (VLAN 10) = 192.168.10.1 /24
* PC3 (VLAN 10) = 192.168.10.2 /24
* PC2 (VLAN 20) = 192.168.20.1 /24
* PC4 (VLAN 20) = 192.168.20.2 /24

---

# 🔹 4. Configuration

### Sur SW1

```bash
enable
configure terminal
!
vlan 10
 name Sales
vlan 20
 name HR
!
interface fa0/1   # PC1
 switchport mode access
 switchport access vlan 10
!
interface fa0/2   # PC2
 switchport mode access
 switchport access vlan 20
!
interface fa0/24   # Lien vers SW2
 switchport mode trunk
 switchport trunk allowed vlan 10,20
```

### Sur SW2

```bash
enable
configure terminal
!
vlan 10
 name Sales
vlan 20
 name HR
!
interface fa0/1   # PC3
 switchport mode access
 switchport access vlan 10
!
interface fa0/2   # PC4
 switchport mode access
 switchport access vlan 20
!
interface fa0/24   # Lien vers SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
```

---

# 🔹 5. Vérification

### Sur les switches

```bash
show vlan brief
```

Doit montrer :

* Fa0/1 dans VLAN 10
* Fa0/2 dans VLAN 20
* Fa0/24 → **pas listé** (car c’est un trunk, pas un access port)

```bash
show interfaces trunk
```

Doit montrer que **Fa0/24 est en mode trunk** et transporte VLAN 10 et 20.

---

# 🔹 6. Tests de ping

✅ Fonctionnent :

* PC1 (192.168.10.1) ↔ PC3 (192.168.10.2)
* PC2 (192.168.20.1) ↔ PC4 (192.168.20.2)

❌ Ne fonctionnent pas :

* PC1 ↔ PC2
* PC1 ↔ PC4
* PC2 ↔ PC3

---

# 🔹 7. Résumé

* Le **trunk** permet de transporter **plusieurs VLANs sur une seule liaison** entre switches.
* Chaque trame Ethernet est **taguée (802.1Q)** avec son VLAN ID → le switch récepteur sait dans quel VLAN la placer.
* Sans trunk → seul **un VLAN** passerait sur le lien inter-switch.

---

👉 Veux-tu que je te fasse une **version TP Packet Tracer** (énoncé + corrigé) que tu pourrais donner à tes apprenants pour tester le ping entre PC1↔PC3 et PC2↔PC4 via trunk ?
