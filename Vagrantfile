# -*- mode: ruby -*-
# vi: set ft=ruby :

#Define que o vagrant não crie o ambiente em parelelo, pois o master deve rodar primeiro que os demais.
ENV['VAGRANT_NO_PARALLEL'] = 'yes'
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

master = {
  "master" => {"memory" => 768, "cpu" => "1" , "image" => "generic/ubuntu2204", "ipvm" => "192.168.122.129", "manager" => "true"},
  "master01" => {"memory" => 768, "cpu" => "1" , "image" => "generic/ubuntu2204", "ipvm" => "192.168.122.130", "slave" => "true"},
  "master02" => {"memory" => 768, "cpu" => "1" , "image" => "generic/ubuntu2204", "ipvm" => "192.168.122.131", "slave" => "true"},
  "node01" => {"memory" => 768, "cpu" => "1" , "image" => "generic/ubuntu2204", "ipvm" => "192.168.122.132"},
  "node02" => {"memory" => 768, "cpu" => "1" , "image" => "generic/ubuntu2204", "ipvm" => "192.168.122.133"}
}

$install_docker_script = <<-'SCRIPT'
    #!/bin/bash
    export DEBIAN_FRONTEND=noninteractive

    # Instala o Docker
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh

    # Instala docker-compose
    sudo curl -fsSL "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose

    # Adiciona usuário vagrant no grupo docker
    sudo usermod -aG docker vagrant
SCRIPT

$manager_script = <<-'SCRIPT'
    #!/bin/bash
    docker swarm init --advertise-addr $1
    docker swarm join-token manager | grep docker > /vagrant/manager.sh
    docker swarm join-token worker | grep docker > /vagrant/worker.sh
SCRIPT

Vagrant.configure("2") do |config|
  master.each do |name, conf|
    config.vm.define        "#{name}" do |machine|
      machine.vm.box      = "#{conf["image"]}"
      machine.vm.hostname = "#{name}"
      machine.vm.network    "private_network", ip: "#{conf["ipvm"]}"
      machine.vm.synced_folder ".", "/vagrant", type: "nfs", nfs_version: 4
      machine.vm.provider :libvirt do |kvm|
        kvm.cpus   = conf["cpu"]
        kvm.memory = conf["memory"]
        kvm.keymap = "pt-br"
        kvm.host   = "localhost"
        kvm.uri    = "qemu:///system"
      end
      machine.vm.provision "shell", inline: $install_docker_script, privileged: true
      if "#{conf["manager"]}" == "true"
        machine.vm.provision "shell" do |s|
          s.inline = $manager_script
          s.args   = "#{conf["ipvm"]}"
        end
      else
        if "#{conf["slave"]}" == "true"
          machine.vm.provision "shell", path: "manager.sh"
        else
          machine.vm.provision "shell", path: "worker.sh"
        end
      end
    end
  end
end