Parfait,
je vais te préparer une **progression de labs Packet Tracer** du **plus simple au plus avancé**, avec **toutes les étapes détaillées**, **les commandes CLI exactes**, et **les schémas de câblage** à suivre.
Je vais te les structurer **lab par lab** avec :

* **Objectif pédagogique**
* **Matériel dans Packet Tracer**
* **Schéma de câblage**
* **Étapes de configuration détaillées**
* **Commandes CLI à taper**
* **Vérification / Tests**

---

## **📅 Jour 1 – Labs de base**

---

### **Lab 1 – Communication simple entre 2 PC**

🎯 **Objectif** : Comprendre l’adressage IP et tester un ping.

📦 **Matériel** :

* 2 PC
* 1 câble croisé (ou droit si "Auto-MDI/MDIX" activé dans Packet Tracer)

🔌 **Câblage** :

* PC0 → PC1 (câble croisé ou droit)

🛠 **Étapes de configuration** :

1. **Attribuer les adresses IP**

   * Sur PC0 :
     `IP: 192.168.1.1 / Masque: 255.255.255.0`
   * Sur PC1 :
     `IP: 192.168.1.2 / Masque: 255.255.255.0`
2. Aucun routeur/switch nécessaire.

📜 **Commandes / Interface Graphique** :

* Dans Packet Tracer : cliquer sur PC → Onglet *Desktop* → *IP Configuration* → entrer IP et masque.

✅ **Vérification** :
Sur PC0 → *Command Prompt* →

```
ping 192.168.1.2
```

Doit répondre.

---

### **Lab 2 – Réseau local avec switch**

🎯 **Objectif** : Comprendre l’utilisation d’un switch et de l’adressage IP.

📦 **Matériel** :

* 3 PC
* 1 switch 2960
* 3 câbles droits

🔌 **Câblage** :

* PC0 → FastEthernet0/1
* PC1 → FastEthernet0/2
* PC2 → FastEthernet0/3

🛠 **Étapes de configuration** :

1. **Attribuer les IP** :

   * PC0 : 192.168.10.1 / 255.255.255.0
   * PC1 : 192.168.10.2 / 255.255.255.0
   * PC2 : 192.168.10.3 / 255.255.255.0
2. Aucun paramétrage CLI sur le switch pour l’instant.

✅ **Vérification** :
Depuis PC0 :

```
ping 192.168.10.2
ping 192.168.10.3
```

---

## **📅 Jour 2 – Routage et VLAN**

---

### **Lab 3 – Routage entre deux réseaux**

🎯 **Objectif** : Utiliser un routeur pour connecter deux réseaux différents.

📦 **Matériel** :

* 2 switches
* 1 routeur 2911
* 4 PC
* Câbles droits

🔌 **Câblage** :

* Switch0 → Routeur (G0/0)
* Switch1 → Routeur (G0/1)
* PC0/PC1 sur Switch0
* PC2/PC3 sur Switch1

🛠 **Étapes CLI** :
**Sur le routeur** :

```
enable
configure terminal
interface g0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit
interface g0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
exit
```

**Sur les PC** :

* Réseau 1 : 192.168.1.x / Passerelle : 192.168.1.1
* Réseau 2 : 192.168.2.x / Passerelle : 192.168.2.1

✅ **Test** :
Ping entre PC0 et PC2 → OK.

---

### **Lab 4 – VLAN et trunk**

🎯 **Objectif** : Segmenter un réseau avec VLANs.

📦 **Matériel** :

* 2 switches
* 4 PC
* 1 câble trunk entre les switches

🔌 **Câblage** :

* Switch0 : PC0 (Fa0/1) VLAN 10, PC1 (Fa0/2) VLAN 20
* Switch1 : PC2 (Fa0/1) VLAN 10, PC3 (Fa0/2) VLAN 20
* Fa0/24 ↔ Fa0/24 (trunk)

🛠 **Commandes CLI** :
Sur chaque switch :

```
enable
configure terminal
vlan 10
 name SALES
vlan 20
 name HR
exit
interface fa0/1
 switchport mode access
 switchport access vlan 10
interface fa0/2
 switchport mode access
 switchport access vlan 20
interface fa0/24
 switchport mode trunk
```

✅ **Test** :
PC0 ping PC2 → OK (même VLAN).
PC0 ping PC3 → FAIL (VLAN différents).

---

## **📅 Jour 3 – Avancé : Inter-VLAN, OSPF, NAT, ACL**

---

### **Lab 5 – Inter-VLAN Routing (Router-on-a-Stick)**

🎯 **Objectif** : Faire communiquer VLANs via un routeur.

📦 **Matériel** :

* 1 routeur
* 1 switch
* 4 PC

🛠 **Commandes** :
Sur le switch :

```
vlan 10
 name SALES
vlan 20
 name HR
interface fa0/1
 switchport mode access
 switchport access vlan 10
interface fa0/2
 switchport mode access
 switchport access vlan 20
interface fa0/24
 switchport mode trunk
```

Sur le routeur :

```
interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
no shutdown
```

✅ **Test** :
PC VLAN 10 ping PC VLAN 20 → OK.

---

### **Lab 6 – NAT avec accès Internet simulé**

🎯 **Objectif** : Simuler accès Internet avec NAT.

📦 **Matériel** :

* 1 routeur
* 1 switch
* 3 PC
* 1 Cloud (Internet simulé)

🛠 **Commandes** :
Sur le routeur :

```
interface g0/0
 ip address 192.168.1.1 255.255.255.0
 ip nat inside
 no shutdown
interface g0/1
 ip address 203.0.113.1 255.255.255.0
 ip nat outside
 no shutdown
ip nat inside source list 1 interface g0/1 overload
access-list 1 permit 192.168.1.0 0.0.0.255
```

