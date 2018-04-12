DEFAULT_BASE_BOX = "bento/centos-7.2"
Vagrant.configure("2") do |config|
  ui = Vagrant::UI::Colored.new
    machines = {    
	:server1 => {:ip => '192.168.56.11', :mem => '512', :cpu => 1},
	:client1 => {:ip => '192.168.56.12', :mem => '512', :cpu => 1},
  }
  config.vm.box = DEFAULT_BASE_BOX

  machines.each do |machine_name, machine_details|
    config.vm.define machine_name do |machine_config|
      machine_ip = machine_details[:ip]
      ui.info("Handling vm with hostname [#{machine_name.to_s}] and IP [#{machine_ip}]")
      machine_config.vm.box = machine_details[:box] ||= DEFAULT_BASE_BOX
      machine_config.vm.network :private_network, ip: machine_ip
      machine_config.vm.hostname = machine_name.to_s
      machine_config.vm.provider :virtualbox do |vb|
        reserved_mem = machine_details[:mem] || default_mem
        reserved_cpu = machine_details[:cpu] || default_cpu
        vb.name = machine_name.to_s
        vb.customize ["modifyvm", :id, "--groups", "/Ansible_Lab"]
        vb.customize ["modifyvm", :id, "--memory", reserved_mem]
        vb.customize ["modifyvm", :id, "--cpus", reserved_cpu]
        vb.gui = false
      end #vb
    end #machine_config
  end #machines

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

end
