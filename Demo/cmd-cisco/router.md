Très bonne question 👍
Dans **Cisco Packet Tracer**, le **routeur** fonctionne avec le système **Cisco IOS** (en mode simulé).
Tu peux donc utiliser une **large palette de commandes** pour configurer, tester et diagnostiquer ton réseau.

---

# 🖥️ Les commandes possibles dans un routeur Cisco (Packet Tracer)

## 🔹 1. **Accès et navigation**

```bash
enable               ! passer en mode privilégié
configure terminal   ! entrer en mode configuration globale
exit                 ! sortir d’un mode
end                  ! revenir directement au mode privilégié
?                    ! afficher l’aide contextuelle
```

---

## 🔹 2. **Gestion des interfaces**

```bash
show ip interface brief     ! voir l’état des interfaces et leurs IP
interface fastEthernet 0/0  ! entrer dans l’interface
ip address 192.168.1.1 255.255.255.0   ! configurer une IP
no shutdown                 ! activer l’interface
shutdown                    ! désactiver l’interface
description Lien_Vers_LAN   ! ajouter une description
```

---

## 🔹 3. **Routage**

### Routage statique

```bash
ip route 192.168.2.0 255.255.255.0 192.168.1.2   ! ajouter une route statique
```

### Routage par défaut

```bash
ip route 0.0.0.0 0.0.0.0 192.168.1.254
```

### Routage dynamique (si activé)

```bash
router rip
 version 2
 network 192.168.1.0
 network 192.168.2.0
 no auto-summary
```

---

## 🔹 4. **Tests & Diagnostics**

```bash
ping 192.168.1.10       ! tester la connectivité
traceroute 192.168.2.10 ! afficher le chemin (dans Packet Tracer, parfois 'tracert')
show running-config     ! voir la config en cours
show startup-config     ! voir la config sauvegardée
show arp                ! voir la table ARP
show mac-address-table  ! voir les adresses MAC connues (plutôt sur un switch)
show protocols          ! vérifier les protocoles activés
```

---

## 🔹 5. **Sécurité de base**

```bash
enable secret cisco         ! mot de passe pour entrer en mode privilégié
line console 0
 password class
 login
exit

line vty 0 4
 password telnetpwd
 login
exit
```

---

## 🔹 6. **Sauvegarde et gestion**

```bash
copy running-config startup-config   ! sauvegarder la config
erase startup-config                 ! effacer la config sauvegardée
reload                               ! redémarrer le routeur
```

---

# ✅ Exemple typique pour ton lab (2 PC, 2 Switchs, 1 Routeur)

```bash
enable
configure terminal

! Interface côté PC1
interface fastEthernet 0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit

! Interface côté PC2
interface fastEthernet 0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
exit

end
write memory
```


