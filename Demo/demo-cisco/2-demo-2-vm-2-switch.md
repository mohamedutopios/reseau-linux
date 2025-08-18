Parfait 👌 tu veux une **démo très simple sans VLAN**, pour montrer le **fonctionnement de base des switches** avec 2 switches reliés et 4 machines.

---

# 🔹 1. Topologie

```
 PC1 ----- SW1 ----- SW2 ----- PC3
 PC2 -----/                 \----- PC4
```

* **SW1** relié à **SW2** par Fa0/24.
* PC1 & PC2 connectés sur SW1.
* PC3 & PC4 connectés sur SW2.

---

# 🔹 2. Adressage IP

On reste dans le même réseau (192.168.1.0/24) :

* PC1 → 192.168.1.1 /24
* PC2 → 192.168.1.2 /24
* PC3 → 192.168.1.3 /24
* PC4 → 192.168.1.4 /24

(Pas besoin de passerelle, communication locale uniquement).

---

# 🔹 3. Configuration

👉 Ici, **aucune config particulière** côté switch.
Par défaut :

* Tous les ports des switches sont en **mode access VLAN 1**.
* Le lien SW1 ↔ SW2 se comporte comme une liaison normale (toutes les machines sont dans le même domaine de broadcast).

Donc en Packet Tracer, il suffit de **câbler et d’attribuer les adresses IP**.

---

# 🔹 4. Déroulé de la communication

Exemple : PC1 veut pinger PC3.

1. PC1 regarde sa table ARP → vide.
2. Il envoie une **requête ARP broadcast** : "Qui a 192.168.1.3 ?"

   * SW1 reçoit la trame → l’inonde sur tous ses ports (sauf l’origine).
   * Elle traverse le lien vers SW2.
   * SW2 inonde la trame sur ses ports (PC3 et PC4).
3. PC3 reconnaît son IP → répond avec son **adresse MAC** à PC1.
4. PC1 et PC3 échangent ensuite directement en unicast → **ping réussi** ✅.

---

# 🔹 5. Vérification sur les switches

Tu peux taper sur chaque switch :

```bash
show mac address-table
```

Après un ping :

* Sur **SW1** : tu verras la MAC de PC1 et PC2 en local, et la MAC de PC3 et PC4 derrière l’interface Fa0/24.
* Sur **SW2** : tu verras la MAC de PC3 et PC4 en local, et celles de PC1 et PC2 derrière Fa0/24.

---

# 🔹 6. Résultat attendu

* ✅ PC1 ↔ PC2 ↔ PC3 ↔ PC4 communiquent sans problème.
* ✅ Tous les PC sont dans le **même réseau IP** et le même domaine de broadcast.
* ❌ Pas d’isolation → si tu envoies un broadcast (ping 192.168.1.255), **tous les PC le reçoivent**.