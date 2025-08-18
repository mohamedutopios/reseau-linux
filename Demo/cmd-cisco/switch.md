Bonne question 👍 !
Quand on apprend Cisco, il faut bien maîtriser les **commandes de base pour les switches**. Voici un récapitulatif clair, avec **mode utilisateur**, **mode privilégié**, **mode configuration**, et les commandes les plus utilisées.

---

# 🔹 1. Modes dans IOS Cisco

* **Mode utilisateur (>)** → commandes simples, pas de config.
* **Mode privilégié (#)** → accès aux commandes avancées (debug, show).
* **Mode configuration (config)#** → modification de la config.
* **Sous-modes** → config d’interface, VLAN, etc.

Exemple :

```
Switch> enable
Switch# configure terminal
Switch(config)# interface fa0/1
Switch(config-if)#
```

---

# 🔹 2. Commandes de navigation

```bash
enable                   # Passer en mode privilégié
disable                  # Revenir au mode utilisateur
configure terminal       # Passer en mode configuration globale
exit                     # Sortir d’un mode
end                      # Quitter directement et revenir en mode privilégié
```

---

# 🔹 3. Commandes de base sur le switch

### Informations système

```bash
show running-config       # Affiche la configuration en cours
show startup-config       # Affiche la configuration sauvegardée (NVRAM)
show version              # Infos IOS et matériel
show vlan brief           # Liste des VLANs et leurs ports
show interfaces status    # État des interfaces
show mac address-table    # Table des adresses MAC
clear mac address-table dynamic 
```

### Sauvegarde & reset

```bash
write memory              # Sauvegarde la config (équivalent copy run start)
copy running-config startup-config   # Sauvegarde manuelle
erase startup-config      # Efface la config sauvegardée
reload                    # Redémarre le switch
```

---

# 🔹 4. Commandes de configuration

### Nom et mot de passe

```bash
hostname SW1
enable password cisco         # Mot de passe simple
enable secret cisco           # Mot de passe chiffré
service password-encryption   # Chiffre tous les mots de passe
```

### Configuration VLAN et ports

```bash
vlan 10
 name Sales
!
interface fa0/1
 switchport mode access
 switchport access vlan 10
!
interface fa0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20
```

### Adresse IP de management

(Nécessaire pour accéder au switch via ping ou Telnet/SSH)

```bash
interface vlan 1
 ip address 192.168.1.2 255.255.255.0
 no shutdown
!
ip default-gateway 192.168.1.1
```

---

# 🔹 5. Commandes de test réseau

```bash
ping 192.168.1.1         # Test de connectivité
traceroute 192.168.1.1   # Tracer la route
```

---

# 🔹 6. Sécurité de base (accès console & VTY)

```bash
line console 0
 password cisco
 login
!
line vty 0 4
 password cisco
 login
 transport input telnet ssh
```

---

Parfait 👍 tu veux un zoom **juste sur la partie MAC Address Table** d’un switch Cisco.
Voici toutes les commandes utiles :

---

# 🔹 1. Afficher la table MAC

```bash
show mac address-table
```

ou version détaillée (selon IOS) :

```bash
show mac address-table dynamic
show mac address-table static
```

📌 Exemple de sortie :

```
Vlan    Mac Address       Type        Ports
----    -----------       ----        -----
   1    00D0.97BF.A1C2    DYNAMIC     Fa0/1
   1    00D0.97BF.A1D4    DYNAMIC     Fa0/2
```

---

# 🔹 2. Vider la table MAC

```bash
clear mac address-table dynamic
```

👉 Efface toutes les adresses dynamiques (le switch devra réapprendre les MACs).

---

# 🔹 3. Supprimer une entrée précise

```bash
clear mac address-table dynamic address 00D0.97BF.A1C2 vlan 1
```

👉 Supprime uniquement la MAC spécifiée pour un VLAN donné.

---

# 🔹 4. Ajouter une entrée statique (manuelle)

```bash
mac address-table static 0011.2233.4455 vlan 10 interface fa0/1
```

👉 Associe **à vie** la MAC `00:11:22:33:44:55` au port `Fa0/1` dans le VLAN 10.
(Utile pour de la sécurité ou du filtrage).

---

# 🔹 5. Supprimer une entrée statique

```bash
no mac address-table static 0011.2233.4455 vlan 10 interface fa0/1
```

---

# 🔹 6. Limiter le nombre de MAC apprises (Port Security)

👉 Même si ce n’est pas **dans la table directement**, souvent on combine avec la sécurité :

```bash
interface fa0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
```

---

✅ En résumé, les **3 commandes clés** pour ton TP sont :

* `show mac address-table` → voir les adresses.
* `clear mac address-table dynamic` → vider la table.
* `mac address-table static ...` → ajouter une entrée fixe.


