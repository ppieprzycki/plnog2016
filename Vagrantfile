# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

$script_vm1 = <<SCRIPT
sudo ip route add 10.0.0.0/8 via 172.16.1.1
SCRIPT


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.ssh.insert_key = false
  config.ssh.username = "vagrant"
 
  required_plugins = %w( vagrant-host-shell vagrant-share vagrant-junos vagrant-vyos )
  required_plugins.each do |plugin|
    system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
  end

  config.vm.define :"juniper1" do |juniper1|
    juniper1.vm.box = "juniper/ffp-12.1X47-D15.4-packetmode"
    juniper1.vm.network "private_network", ip: "192.168.56.11"
    juniper1.vm.network "private_network", ip: "172.168.1.1", virtualbox__intnet: "network1"
    juniper1.vm.network "private_network", ip: "10.0.12.1", virtualbox__intnet: "network2"
    juniper1.vm.network "private_network", ip: "10.0.13.1", virtualbox__intnet: "network3"
  end

 

  config.vm.define :"vyos2" do |vyos2|
    vyos2.vm.box = "higebu/vyos-1.1.7-amd64"
    vyos2.vm.network "private_network", ip: "10.0.12.2", virtualbox__intnet: "network2"
  end

  config.vm.define :"vyos3" do |vyos3|
    vyos3.vm.box = "higebu/vyos-1.1.7-amd64"
    vyos3.vm.network "private_network", ip: "10.0.13.3", virtualbox__intnet: "network3"
  end


  config.vm.define :linux1 do |linux1|
    linux1.vm.box = "ubuntu/trusty64"
    linux1.vm.network "private_network", ip: "172.16.1.100", virtualbox__intnet: "network1"
    linux1.vm.provision "shell",
      run: "always",
      inline: $script_vm1
    linux1.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
  end
 

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.groups = {
      "civyos" => ["vyos2", "vyos3"],
      "cijun" => ["juniper1"],
      "citest:children" => ["civyos", "cijun"]
    }
    ansible.host_vars = {
      "juniper1" => {"ansible_ssh_user" => "root",
                  "ansible_ssh_password" => "Juniper"}
    }
    ansible.verbose = "v"
  end
end

