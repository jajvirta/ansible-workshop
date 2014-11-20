# -*- coding: utf-8 -*-
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = "hashicorp/precise64"

  config.vm.provision "shell", path: "bootstrap.sh"
  config.vm.synced_folder ".", "/vagrant", :owner=> 'vagrant', :mount_options => ['dmode=755', 'fmode=644']

#  config.vm.network :private_network, ip: "10.30.30.2"
#  config.vm.network "forwarded_port", guest: 7210, host: 7210
#  config.vm.network "forwarded_port", guest: 80, host: 80

  config.vm.provider "virtualbox" do |v|
    v.memory = 2000
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end
end
