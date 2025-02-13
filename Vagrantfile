# -*- mode: ruby -*-
# vi: set ft=ruby :

nodes = [
  { :hostname => 'swarm-master-1', :ip => '192.168.77.10', :ram => 2048, :cpus => 1 },
  { :hostname => 'swarm-master-2', :ip => '192.168.77.11', :ram => 2048, :cpus => 1 },
  { :hostname => 'swarm-worker-1', :ip => '192.168.77.12', :ram => 4096, :cpus => 2 },
  { :hostname => 'swarm-worker-2', :ip => '192.168.77.13', :ram => 4096, :cpus => 2 }
]

Vagrant.configure("2") do |config|
  # vagrant-hostmanager options
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = false
  # Always use Vagrant's insecure key
  #config.ssh.insert_key = false
  # Forward ssh agent to easily ssh into the different machines
  config.ssh.forward_agent = true
  # Vagrant box
  config.vm.box = "bento/ubuntu-20.04"
  # Docker
  config.vm.provision "docker"
  # Synced Folder
  config.vm.synced_folder '.', '/vagrant', disabled: true

  # Provision nodes
  nodes.each do |node|
    config.vm.define node[:hostname] do |config|
      config.vm.hostname = node[:hostname]
      config.vm.network :private_network, ip: node[:ip]

      memory = node[:ram] ? node[:ram] : 2048;
      cpus = node[:cpus] ? node[:cpus] : 1;

      config.vm.provider :virtualbox do |vb|
        vb.customize [
          "modifyvm", :id,
          "--memory", memory.to_s,
          "--cpus", cpus.to_s
        ]
      end

      # Provision both VMs using Ansible after the last VM is booted.
      if node[:hostname] == "swarm-worker-2"
        config.vm.provision "ansible" do |ansible|
          ansible.compatibility_mode = "2.0"
          ansible.playbook = "ansible/playbook.yml"
          ansible.inventory_path = "ansible/inventory"
          ansible.limit = "all"
        end
      end
    end
  end
end
