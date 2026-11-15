Vagrant.configure("2") do |config|
  # Définition du nœud Web
  config.vm.define "web-node" do |web|
    web.vm.box = "debian/bookworm64"
    web.vm.network "private_network", ip: "192.168.56.11"
    web.vm.provider "virtualbox" do |vb|
      vb.memory = "512" # 512 Mo suffisent largement pour Apache
      vb.cpus = 1
      vb.gui = false
    end
  end

  # Définition du nœud DB
  config.vm.define "db-node" do |db|
    db.vm.box = "debian/bookworm64"
    db.vm.network "private_network", ip: "192.168.56.12"
    db.vm.provider "virtualbox" do |vb|
      vb.memory = "512" # 512 Mo pour une base PostgreSQL vide
      vb.cpus = 1
      vb.gui = false
    end
  end

  # Définition du nœud de contrôle Ansible
  config.vm.define "control-node" do |ctrl|
    ctrl.vm.box = "debian/bookworm64"
    ctrl.vm.network "private_network", ip: "192.168.56.10"
    ctrl.vm.provider "virtualbox" do |vb|
      vb.memory = "2048" # 1 Go car Ansible a besoin d'un peu plus de RAM pour s'exécuter
      vb.cpus = 1
      vb.gui = false
    end
  end
end