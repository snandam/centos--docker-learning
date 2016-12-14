# -*- mode: ruby -*-
# vi: set ft=ruby :

#Set the default provider to virtualbox
# https://docs.vagrantup.com/v2/providers/default.html
# http://www.vagrantbox.es/
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|


  config.vm.box = "centos/7"
  #Vagrant box with puppet preinstalled
  #config.vm.box_url = "http://puppet-vagrant-boxes.puppetlabs.com/centos-65-x64-virtualbox-puppet.box"


  #Sync the current directory to /vagrant
  config.vm.synced_folder ".", "/vagrant"

  #Networking between the host and vagrant box

  #Forward the port only
  #config.vm.network "forwarded_port", guest: 80, host: 1234

  #Make it look like another machine accessible only by the host
  config.vm.network "private_network", ip: "192.168.50.4"

  # vagrant provision --provision-with ansible
  config.vm.provision "ansible" do |ansible|

  ansible.playbook = "provisioning/ansible/playbooks/install-docker-centos/main.yml"

  end
end
