Parfait 👍 ! Tu as tout à fait raison : si on veut **convaincre de l’intérêt des VLANs**, rien de mieux qu’une démo **avec adresses IP** → comparer **sans VLAN** et **avec VLAN**.

---

# 🔹 1. Topologie de base

```
PC1 ---- SW1 ---- PC2
PC3 ---- SW1 ---- PC4
```

### Adressage

* PC1 = 192.168.10.1 /24
* PC2 = 192.168.10.2 /24
* PC3 = 192.168.20.3 /24
* PC4 = 192.168.20.4 /24

⚠️ Pas de gateway (on reste en réseau local).

---

# 🔹 2. Cas A : Sans VLAN (par défaut VLAN 1)

### Configuration du switch

Pas besoin de config particulière → tout est dans **VLAN 1**.

### Expérience

* PC1 envoie un **ping vers 192.168.10.255**.

  * ping 192.168.10.255 -n 1
  * Avant le ping, PC1 envoie un **ARP broadcast** : "Qui a 192.168.10.255 ?".
  * Le switch **inonde le broadcast sur tous les ports**.
  * PC3 reçoit → répond avec son MAC.
  * Le ping fonctionne ✅.

👉 Résultat : **tous les PC sont dans le même domaine de broadcast**.
Un broadcast part de PC1 et atteint PC2, PC3, PC4.

---

# 🔹 3. Cas B : Avec VLANs

### Configuration du switch

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
 switchport access vlan 10
!
interface fa0/3   # PC3
 switchport mode access
 switchport access vlan 20
!
interface fa0/4   # PC4
 switchport mode access
 switchport access vlan 20
```

### Expérience

* PC1 ping **PC2 (192.168.10.2)** → fonctionne ✅

  * Car ils sont tous les deux en VLAN 10.
* PC1 ping **PC3 (192.168.20.3)** → échoue ❌

  * PC1 envoie une requête ARP broadcast "Qui a 192.168.10.3 ?"
  * Le switch diffuse ce broadcast uniquement dans **VLAN 10**.
  * PC3 est dans **VLAN 20** → il ne reçoit jamais la question.
  * Pas de réponse ARP → pas de ping.

👉 Résultat : **les broadcasts ne traversent pas les VLANs**.

---

# 🔹 4. Résumé clair pour la démo

* **Sans VLAN** :

  * Ping de PC1 → PC3 fonctionne.
  * Tous les PC reçoivent les broadcasts.
  * Un seul domaine de broadcast → réseau vite saturé si beaucoup de machines.

* **Avec VLANs** :

  * PC1 communique seulement avec PC2 (même VLAN).
  * PC1 ne peut pas parler à PC3 (VLAN différent).
  * Broadcast limité → segmentation et sécurité accrues.

---

👉 Tu veux que je te fasse un **scénario complet de TP Packet Tracer** avec **commandes + tests ping + show vlan brief** (comme si tu le donnais à des étudiants) pour comparer **Cas A** et **Cas B** ?


