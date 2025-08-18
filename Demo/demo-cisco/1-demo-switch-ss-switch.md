Parfait 👌 tu veux des **démos pratiques dans Packet Tracer**, ciblées **couche 2 (liaison)**. Je vais te préparer un **TP progressif** avec étapes détaillées, commandes CLI, et ce que tu pourras observer (tables MAC, ping, VLAN).

---

# 🔹 1. **Faire communiquer 2 machines (PC ↔ PC en direct)**

### Topologie :

```
PC1 -------- PC2
```

### Étapes :

1. Placer 2 PC.
2. Relier avec un câble **crossover** (dans Packet Tracer, un câble cuivre direct fonctionne aussi).
3. Configurer IP :

   * PC1 : `192.168.1.1 /24`
   * PC2 : `192.168.1.2 /24`
4. Tester la communication :

   ```
   PC1> ping 192.168.1.2
   ```

   ✅ Résultat : succès.

👉 **Notion L2/L3 :** ARP sera utilisé pour trouver l’adresse MAC de l’autre PC.

---

# 🔹 2. **Tester la notion de sous-réseau**

### Topologie :

```
PC1 -------- PC2
```

### Étapes :

1. Configurer IP :

   * PC1 : `192.168.1.1 /24`
   * PC2 : `192.168.2.1 /24`
2. Tester :

   ```
   PC1> ping 192.168.2.1
   ```

   ❌ Résultat : échec (ils sont dans 2 sous-réseaux différents → pas de routeur).

👉 **Notion :** au niveau L2 ils peuvent échanger ARP, mais au L3 la communication échoue car **pas dans le même sous-réseau**.

---

# 🔹 3. **Installer un switch et faire communiquer des machines**

### Topologie :

```
PC1 ----+
        |
       [Switch]
        |
PC2 ----+
```

### Étapes :

1. Ajouter un switch (2960).
2. Relier les PC au switch avec câbles droits.
3. Configurer IP :

   * PC1 : `192.168.1.1 /24`
   * PC2 : `192.168.1.2 /24`
4. Test :

   ```
   PC1> ping 192.168.1.2
   ```

   ✅ Résultat : succès.

👉 **Notion :** le switch apprend dynamiquement les **adresses MAC** et construit sa table CAM.

