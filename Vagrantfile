# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # General Vagrant VM configuration.
  config.vm.box = "debian/bookworm64"
  config.ssh.insert_key = false
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.provider :virtualbox do |v|
    v.memory = 512
    v.linked_clone = true
    v.check_guest_additions = false
  end

  config.trigger.before :"Vagrant::Action::Builtin::WaitForCommunicator", type: :action do|t|
    t.warn = "Iterrompe o servidor dhcp do virtualbox"
    t.run = {inline: "vboxmanage dhcpserver stop --interface vboxnet0"}
  end

  # Configuração comum a todas as máquinas (roda antes da configuração de cada uma).
  config.vm.provision "ansible" do |ansible|
    ansible.compatibility_mode = "2.0"
    ansible.playbook = "playbooks/comum.yml"
  end

  # File server (DHCP, DNS, LVM e NFS). Precisa ser a primeira, pois as outras dependem do DHCP.
  config.vm.define "arq" do |arq|
    arq.vm.hostname = "arq.rychardson.zafler.devops"
    arq.vm.network :private_network, ip: "192.168.56.126"
    (0..2).each do |x|
      arq.vm.disk :disk, size: "10GB", name: "disk-#{x}"
    end
    arq.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/arq.yml"
    end
  end

  # Database server.
  config.vm.define "db" do |db|
    db.vm.hostname = "db.rychardson.zafler.devops"
    db.vm.network :private_network, mac: "0800273A0001", type: "dhcp"
    db.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/db.yml"
    end
  end

  # Application server.
  config.vm.define "app" do |app|
    app.vm.hostname = "app.rychardson.zafler.devops"
    app.vm.network :private_network, mac: "0800273A0002", type: "dhcp"
    app.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/app.yml"
    end
  end

  # Client.
  config.vm.define "cli" do |cli|
    cli.vm.hostname = "cli.rychardson.zafler.devops"
    cli.vm.network :private_network, type: "dhcp"
    cli.vm.provider :virtualbox do |v|
      v.memory = 1024
    end
    cli.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/cli.yml"
    end
  end
end
