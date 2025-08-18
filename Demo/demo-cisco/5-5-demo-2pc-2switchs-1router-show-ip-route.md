Parfait 👌 tu viens d’afficher la **table de routage du routeur** avec `show ip route`.

Décryptons ton résultat en détail :

---

# 📖 Lecture de la table de routage

```
Codes: L - local, C - connected, S - static, R - RIP, ...
```

👉 Ce sont les **codes de type de routes** (Connected, Local, Static, etc.).

---

## 🔹 Partie importante

```
Gateway of last resort is not set
```

➡️ Cela signifie que **tu n’as pas configuré de route par défaut** (`ip route 0.0.0.0 0.0.0.0 …`).
Donc si le routeur reçoit un paquet vers un réseau qu’il ne connaît pas, il le **jette**.

---

## 🔹 Réseaux connus

### 192.168.1.0/24

```
C       192.168.1.0/24 is directly connected, GigabitEthernet0/0
L       192.168.1.1/32 is directly connected, GigabitEthernet0/0
```

* `C` → le réseau **192.168.1.0/24** est connecté directement à l’interface **G0/0**.
* `L` → l’adresse **192.168.1.1/32** est l’IP **locale** du routeur sur G0/0.

---

### 192.168.2.0/24

```
C       192.168.2.0/24 is directly connected, GigabitEthernet0/1
L       192.168.2.1/32 is directly connected, GigabitEthernet0/1
```

* `C` → le réseau **192.168.2.0/24** est directement connecté à **G0/1**.
* `L` → l’adresse **192.168.2.1/32** est l’IP **locale** du routeur sur G0/1.

---

# ✅ Conclusion

👉 Ton routeur connaît **deux réseaux directement connectés** :

* **192.168.1.0/24** (via G0/0)
* **192.168.2.0/24** (via G0/1)

Il sait donc **router entre eux** (si les PC des deux réseaux ont bien ce routeur comme **passerelle par défaut**).

⚠️ Par contre, il **ne connaît rien au-delà** → d’où l’absence de **default route** (`Gateway of last resort not set`).

---

Veux-tu que je te montre comment ajouter une **route par défaut** + tester avec un ping entre deux réseaux via ce routeur dans **Packet Tracer** ? 🚀
