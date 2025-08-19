Top — voilà un **mémo des commandes Vagrant utiles** pour ton Vagrantfile (3 VMs `vm1`, `vm2`, `vm3` sur le réseau privé `10.10.10.0/24` via VirtualBox).

# Démarrage / arrêt / état

```bash
vagrant up                 # démarre toutes les VMs
vagrant up vm1             # démarre seulement vm1
vagrant status             # état des VMs du dossier
vagrant halt               # arrête proprement toutes
vagrant halt vm2           # arrête vm2
vagrant suspend vm3        # met vm3 en veille
vagrant resume vm3         # reprend vm3
vagrant destroy -f         # supprime toutes (disques inclus)
vagrant destroy -f vm1     # supprime vm1
```

# Connexion & accès

```bash
vagrant ssh vm1            # SSH direct dans vm1 (user vagrant)
vagrant ssh-config vm2     # affiche la config SSH (hôte/port/clé)
vagrant global-status      # liste toutes les VMs Vagrant sur la machine
```

# (Re)configuration & provisionnement

```bash
vagrant reload             # redémarre et recharge la config
vagrant reload vm2         # idem pour vm2
vagrant reload --provision # reload + exécute provisioners
vagrant provision vm3      # exécute uniquement les provisioners sur vm3
```

# Boxes / plugins

```bash
vagrant box list           # boxes disponibles en local
vagrant box add ubuntu/jammy64          # ajoute une box (si pas déjà)
vagrant box update         # met à jour la box utilisée par ce projet
vagrant plugin install vagrant-vbguest  # (recommandé) VirtualBox Guest Additions
```

# Réseau & infos pratiques (spécifique à ton lab)

* Ton fichier crée un réseau privé **host-only** `10.10.10.0/24`, IPs :

  * `vm1` = **10.10.10.11**
  * `vm2` = **10.10.10.12**
  * `vm3` = **10.10.10.13**
* Test entre VMs :

  ```bash
  vagrant ssh vm1
  ping -c 1 10.10.10.12
  ping -c 1 10.10.10.13
  ```
* Si tu modifies le réseau dans le Vagrantfile → applique :

  ```bash
  vagrant reload    # ou vagrant reload --provision
  ```
* Sur Windows/macOS, si le réseau privé ne se crée pas : vérifie l’adaptateur **VirtualBox Host-Only** (dans VirtualBox > Outils > Réseau).

# Snapshots (points de restauration)

```bash
vagrant snapshot save vm1 pre-test     # crée un snapshot
vagrant snapshot list vm1              # liste
vagrant snapshot restore vm1 pre-test  # restaure
```

# Dossiers partagés & sync

* Par défaut, le dossier projet hôte est monté dans la VM à **/vagrant**.
* Pour forcer la resynchro (si tu utilises rsync) :

  ```bash
  vagrant rsync vm2
  ```

  (utile seulement si tu as défini `config.vm.synced_folder` en mode rsync)

# Nettoyage / dépannage rapide

```bash
vagrant reload vm1
vagrant halt vm1 && vagrant up vm1
vagrant destroy -f vm2 && vagrant up vm2
vagrant box prune          # supprime les anciennes versions de boxes
```

# Astuces

* **Changer le nom VirtualBox** : tu l’as déjà fait (`vb.name = "lab-vmX"`), pratique pour retrouver les VMs dans VirtualBox GUI.
* **Réseau Internet** : la **carte 1 NAT** reste par défaut → `apt`/`curl` fonctionnent sans config.
* **DNS hostnames** entre VMs : utilise directement les IPs (10.10.10.x) ou ajoute des entrées dans `/etc/hosts` de chaque VM si tu veux `vm1`, `vm2`, etc.

Si tu veux, je peux te générer un **Vagrantfile avec provision Ansible/bash** pour auto-installer des paquets (nginx, tools, etc.) au `vagrant up`.
