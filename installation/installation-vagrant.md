Oui, tout à fait. Voici un **Vagrantfile minimal** qui crée **3 VMs Ubuntu** reliées par un **réseau privé** commun. Elles auront des **IP statiques** pour que tu puisses vérifier la communication (ping/SSH). Tu pourras ensuite **retirer les IP statiques** et mettre en place **ton propre DHCP** sur le même réseau.

```ruby
# Vagrantfile - 3 VMs reliées sur un réseau privé "labnet"
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"  # Ubuntu 22.04

  # Réseau de lab (host-only / private_network)
  # NAT (carte 1) reste pour l'accès Internet/apt/SSH vagrant.
  LABNET_NET = "10.10.10"
  LABNET_MASK = "255.255.255.0"

  # --- VM 1 ---
  config.vm.define "vm1" do |vm|
    vm.vm.hostname = "vm1"
    vm.vm.network :private_network, ip: "#{LABNET_NET}.11", netmask: LABNET_MASK
    vm.vm.provider "virtualbox" do |vb|
      vb.name = "lab-vm1"
      vb.memory = 512
      vb.cpus = 1
    end
  end

  # --- VM 2 ---
  config.vm.define "vm2" do |vm|
    vm.vm.hostname = "vm2"
    vm.vm.network :private_network, ip: "#{LABNET_NET}.12", netmask: LABNET_MASK
    vm.vm.provider "virtualbox" do |vb|
      vb.name = "lab-vm2"
      vb.memory = 512
      vb.cpus = 1
    end
  end

  # --- VM 3 ---
  config.vm.define "vm3" do |vm|
    vm.vm.hostname = "vm3"
    vm.vm.network :private_network, ip: "#{LABNET_NET}.13", netmask: LABNET_MASK
    vm.vm.provider "virtualbox" do |vb|
      vb.name = "lab-vm3"
      vb.memory = 512
      vb.cpus = 1
    end
  end
end
```

### Utilisation rapide

1. Sauvegarde ce fichier en `Vagrantfile` dans un dossier vide.
2. Lance : `vagrant up`
3. Teste depuis chaque VM :

```bash
vagrant ssh vm1
ping -c2 10.10.10.12
ping -c2 10.10.10.13
```

### Quand tu voudras passer au DHCP

* Garde la même topologie, mais supprime les IP statiques dans le Vagrantfile (remplace chaque ligne `private_network, ip: ...` par `private_network, dhcp: true`) **OU** laisse Vagrant gérer l’interface mais bascule la config en DHCP côté OS (netplan).
* Installe ton serveur DHCP sur l’une des VMs et configure sa plage (par ex. `10.10.10.100–200`) et la passerelle si besoin.
* Pense à définir le DNS (soit externe, soit celui que tu déploieras).

Si tu veux une variante avec **une seconde carte dédiée au lab** (NAT + lab), je te la mets aussi.
