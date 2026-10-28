# -*- mode: ruby -*-
NOME1   = "rychardson"
NOME2   = "zafler"
DOMINIO = "#{NOME1}.#{NOME2}.devops"
IP_ARQ  = "192.168.56.126"
MAC_DB  = "080027000127"
MAC_APP = "080027000167"

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  config.ssh.insert_key = false                         # geração de chaves desativada
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 512
    vb.linked_clone = true
    vb.check_guest_additions = false
  end

  # Gatilho: desabilita o DHCP do VirtualBox (tenta as sintaxes do VBox 7 e 6)
  config.trigger.before :"VagrantPlugins::ProviderVirtualBox::Action::Boot", type: :action do |t|
    t.name = "Desabilitando o servidor DHCP do VirtualBox"
    t.ruby do |_env, _machine|
      [["--network", "HostInterfaceNetworking-vboxnet0"], ["--interface", "vboxnet0"],
       ["--netname", "HostInterfaceNetworking-vboxnet0"], ["--ifname", "vboxnet0"]].each do |args|
        break if system("VBoxManage", "dhcpserver", "remove", *args, out: File::NULL, err: File::NULL)
      end
    end
  end

  config.vm.define "arq" do |m|
    m.vm.hostname = "arq.#{DOMINIO}"
    m.vm.network "private_network", ip: IP_ARQ
    m.vm.network "forwarded_port", guest: 22, host: 2221, host_ip: "127.0.0.1", id: "ssh"
    (1..3).each { |i| m.vm.disk :disk, size: "10GB", name: "dados#{i}" }   # 3 discos de 10 GB
  end

  config.vm.define "db" do |m|
    m.vm.hostname = "db.#{DOMINIO}"
    m.vm.network "private_network", type: "dhcp", ip: "192.168.56.1", mac: MAC_DB
    m.vm.network "forwarded_port", guest: 22, host: 2222, host_ip: "127.0.0.1", id: "ssh"
  end

  config.vm.define "app" do |m|
    m.vm.hostname = "app.#{DOMINIO}"
    m.vm.network "private_network", type: "dhcp", ip: "192.168.56.1", mac: MAC_APP
    m.vm.network "forwarded_port", guest: 22, host: 2223, host_ip: "127.0.0.1", id: "ssh"
  end

  config.vm.define "cli" do |m|
    m.vm.hostname = "cli.#{DOMINIO}"
    m.vm.network "private_network", type: "dhcp", ip: "192.168.56.1"
    m.vm.network "forwarded_port", guest: 22, host: 2224, host_ip: "127.0.0.1", id: "ssh"
    m.vm.provider("virtualbox") { |vb| vb.memory = 1024 }                 # cli com 1024 MB
  end
end
