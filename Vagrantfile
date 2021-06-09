# -*- mode: ruby -*-
# vi: set ft=ruby noet :

# Require YAML module
require 'yaml'
ansible_hosts = YAML.load_file(File.join(File.dirname(__FILE__), 'inventory/vagrant/hosts.yml'))
machines = ansible_hosts["all"]["hosts"]

Vagrant.configure("2") do |config|
  ui = Vagrant::UI::Colored.new
  
  # Faster option no need to be secure on local only VM
  config.ssh.insert_key = false
  config.ssh.private_key_path = File.expand_path("~/.vagrant.d/insecure_private_key")

  # Save time for not checking updates
  config.vm.box_check_update = false

  SharedFoldersEnableSymlinksCreate = false

  machines.each do |machine_name, machine_details|
    config.vm.define machine_name do |machine_config|
      machine_ip = machine_details["ansible_host"]
      ui.info("Handling vm with hostname [#{machine_name.to_s}] and IP [#{machine_ip}]")
      machine_config.vm.box = machine_details[:box] ||= ansible_hosts["all"]["vars"]["default_base_box"]
      machine_config.vm.network :private_network, ip: machine_ip
      machine_config.vm.hostname = machine_name.to_s
      # Disabling sync folder
      machine_config.vm.synced_folder '.', '/vagrant', disabled: true
      machine_config.vm.provider :virtualbox do |vb|
        # Save time
        vb.check_guest_additions = false
        reserved_mem = machine_details["mem"] || ansible_hosts["all"]["vars"]["default_mem"]
        reserved_cpu = machine_details["cpu"] || ansible_hosts['all']["vars"]["default_cpu"]
        vb.name = machine_name.to_s
        vb.customize ["modifyvm", :id, "--groups", "/Ansible_Lab"]
        vb.customize ["modifyvm", :id, "--memory", reserved_mem]
        vb.customize ["modifyvm", :id, "--cpus", reserved_cpu]
        # Workarround for VPN which change name resolution on host
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
        vb.gui = false
      end #vb
    end #machine_config
  end #machines

# Not creating ACONTROLLER VM if no need for it
if ENV["ACONTROLLER"]
  config.vm.define 'acontroller' do |machine|
    machine.vm.synced_folder "./", "/vagrant", owner: "vagrant", mount_options: ["dmode=775,fmode=600"]
    machine.vm.network "private_network", ip: "192.168.56.2"
	machine.vm.provider :virtualbox do |vb|
      vb.name = "acontroller"
      vb.customize ["modifyvm", :id, "--groups", "/Ansible_Lab"]
      vb.customize ["modifyvm", :id, "--memory", 512]
      vb.customize ["modifyvm", :id, "--cpus", 1]
      vb.gui = false
    end
	machine.vm.provision "file", source: "~/.gitconfig", destination: ".gitconfig"
	machine.vm.provision "file", source: "dev-settings/screenrc", destination: ".screenrc"
	machine.vm.provision "file", source: "dev-settings/vimrc", destination: ".vimrc"
	machine.vm.provision :shell do |s|
	  s.inline = "sudo yum install epel-release -y \
	    && sudo yum install python-pip -y \
	    && sudo yum -y install screen vim git \
	    && sudo pip install ansible==2.4.2.0 \
	    && cd /vagrant/ \
	    && ansible-galaxy install -r requirements.yml"
	end
    machine.vm.provision :ansible_local do |ansible|
      ansible.playbook       = "site.yml" # can be deploy.yml
      ansible.verbose        = false # can be "-vvv"
      ansible.install        = false
      ansible.limit          = "all" # or only "nodes" group, etc.
      ansible.inventory_path = "inventory/vagrant/"
    end
  end
end #if ENV["ACONTROLLER"]
end
