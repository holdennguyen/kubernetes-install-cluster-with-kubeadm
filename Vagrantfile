# -*- mode: ruby -*-
# vi:set ft=ruby sw=2 ts=2 sts=2:

# Define the number of control plane (MASTER_NODE) and node (WORKER_NODE)
NUM_MASTER_NODE = 1
NUM_WORKER_NODE = 2

IP_NW = "192.168.56."
MASTER_IP_START = 1
NODE_IP_START = 2

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  # Here are some key details about the "ubuntu/bionic64" Vagrant box:
    # Operating System: Ubuntu 18.04 LTS (Bionic Beaver)
        # Ubuntu 18.04 LTS will receive security updates and bug fixes 
        # from Canonical, the company behind Ubuntu, until April 2023 
        # for desktop and server versions, and until April 2028 for 
        # server versions with Extended Security Maintenance (ESM) enabled.
    # Architecture: x86_64 (64-bit)
    # Disk Size: 10 GB
    # RAM: 2 GB
    # CPUs: 2
    # Desktop Environment: None (headless)
    # Provider: VirtualBox
  config.vm.box = "ubuntu/bionic64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

  # View the documentation for the VirtualBox for more
  # information on available options.
  # https://developer.hashicorp.com/vagrant/docs/providers/virtualbox/configuration

  # Provision Control Plane
  (1..NUM_MASTER_NODE).each do |i|
      config.vm.define "kubemaster" do |node|
        node.vm.provider "virtualbox" do |vb|
            vb.name = "kubemaster"
            vb.memory = 2048
            vb.cpus = 2
        end
        node.vm.hostname = "kubemaster"
        node.vm.network :private_network, ip: IP_NW + "#{MASTER_IP_START + i}"
      end
  end


  # Provision Nodes
  (1..NUM_WORKER_NODE).each do |i|
    config.vm.define "kubenode0#{i}" do |node|
        node.vm.provider "virtualbox" do |vb|
            vb.name = "kubenode0#{i}"
            vb.memory = 2048
            vb.cpus = 2
        end
        node.vm.hostname = "kubenode0#{i}"
        node.vm.network :private_network, ip: IP_NW + "#{NODE_IP_START + i}"
    end
  end
end