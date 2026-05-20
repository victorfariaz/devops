# -*- mode: ruby -*-
# vi: set ft=ruby :

## Configurações globais das VMs:

# Definir o provider como VirtualBox
# Definir a imagem base como debian/bookworm64
# Desativar a geração automática de chaves ssh pelo Vagrant

Vagrant.configure("2") do |config|
  config.vm.provider 'virtualbox'
  config.vm.box = 'debian/bookworm64'
  config.ssh.insert_key = false

# Criar gatilho para desabilitar DHCP do VirtualBox
  config.trigger.before :"Vagrant::Action::Builtin::WaitForCommunicator", type: :action do |t|
    t.warn = "Desabilitando servidor DHCP do VirtualBox"
    t.run = {inline: "VBoxManage dhcpserver stop --interface vboxnet0"}
  end

 # Desabilitar verificação/atualização automática do Guest Additions
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end
  
  
## Configurações do provider para todas as VMs:
 
# Definir memória RAM de 512 MB para todas as VMs (será sobrescrito para cli)
# Habilitar clones vinculados para economizar espaço em disco
# Desabilitar verificação/atualização automática do Guest Additions

  config.vm.provider 'virtualbox' do |v|
    v.memory = '512'
    v.linked_clone = true
    v.check_guest_additions = false
  end


## Playbook com configurações comuns (executa em todas as VMs):

  config.vm.provision "ansible" do |ansible|
    ansible.compatibility_mode = "2.0"
    ansible.playbook = "ansible/playbooks/common.yml"
    ansible.groups = {
      "all" => ["arq", "db", "app", "cli"]
    }
  end


## Configurações do servidor de arquivos [arq]:

# Definir hostname
# Configurar endereço IP estático
# Criar 3 discos adicionais de 10 GB usando vm.disk

    config.vm.define "arq" do |arq|
      arq.vm.hostname = "arq.victor.devops"
      arq.vm.network :private_network, ip: "192.168.56.114"
      
      (1..3).each do |i|
        arq.vm.disk :disk, size: "10GB", name: "arq-disk-#{i}"
      end

# Configurar DHCP e DNS no servidor de arquivos
      
      arq.vm.provision "ansible" do |ansible|
        ansible.compatibility_mode = "2.0"
        ansible.playbook = "ansible/playbooks/arq-dhcp-dns.yml"
      end
    
# Configurar LVM e NFS no servidor de arquivos
      arq.vm.provision "ansible" do |ansible|
        ansible.compatibility_mode = "2.0"
        ansible.playbook = "ansible/playbooks/arq-lvm-nfs.yml"
      end
    end

## Configurações do servidor de banco de dados [db]:

# Definir hostname
# Configurar rede para obter IP via DHCP com reserva por MAC
# Configurar MAC address para reserva estática

  config.vm.define "db" do |db|
    db.vm.hostname = "db.victor.devops"
    db.vm.network :private_network,
      type: "dhcp",
      mac: "080027000001"
    
# Configurar MariaDB e NFS no servidor de banco de dados
    db.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "ansible/playbooks/db.yml"
    end
  end


## Configurações do servidor de aplicação [app]:

# Definir hostname
# Configurar rede para obter IP via DHCP com reserva por MAC
# Configurar MAC address para reserva estática

  config.vm.define "app" do |app|
    app.vm.hostname = "app.victor.devops"
    app.vm.network :private_network,
      type: "dhcp",
      mac: "080027000002"
    
# Configurar Apache e NFS no servidor de aplicação
    app.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "ansible/playbooks/app.yml"
    end
  end


## Configurações do host cliente [cli]:

# Definir hostname
# Configurar rede para obter IP via DHCP (dinâmico)
# Aumentar memória RAM para 1024 MB

  config.vm.define "cli" do |cli|
    cli.vm.hostname = "cli.victor.devops"
    cli.vm.network :private_network, type: "dhcp"
    
    
    cli.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
    end
    
# Configurar Firefox, X11 e NFS no host cliente
    cli.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "ansible/playbooks/cli.yml"
    end
  end
end
