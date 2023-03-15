# Vagrant
<p align="left">
<img src="/docs/images/vagrant-logo.png" width="50">
</p>


`Vagrant` is a tool for creating and managing virtual machine environments. It is often considered a form of `Infrastructure as Code (IaC)`, as it allows us to define and manage our infrastructure using code. However, it is important to note that Vagrant is primarily a tool for managing virtual machine environments, and is not typically used for managing production infrastructure.

## Quick Start

For the quick-start, we'll bring up virtual machines on `VirtualBox` because it is free and works on all major platforms.

First, download and install the [VirtualBox](https://www.virtualbox.org/wiki/Download_Old_Builds), [Vagrant](https://www.vagrantup.com/downloads.html) on your host.

We will build our virtual environment from `Vagrantfile`. Clone this repo or just copy this [Vagrantfile](../../Vagrantfile) to your current directory.

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

In this Vagrantfile, we simply specify: 
- Number of virtual machines: `NUM_MASTER_NODE`, `NUM_WORKER_NODE`
- IP address: `IP_NW`, `MASTER_IP_START`, `NODE_IP_START`
- Private networking connectivity: `node.vm.network`
- Unique hostname: `node.vm.hostname`
- Operating system: `config.vm.box`
- System resources: `vb.memory`, `vb.cpus`

The syntax of Vagrantfiles is [Ruby](https://www.ruby-lang.org/en/), but knowledge of the Ruby programming language is not necessary to make modifications to the Vagrantfile. See [here](https://developer.hashicorp.com/vagrant/docs/vagrantfile) for more information about Vagrantfile syntax.

## Start provisioning

Run the command:

    vagrant up

Output should be simillar:

    Bringing machine 'kubemaster' up with 'virtualbox' provider...
    Bringing machine 'kubenode01' up with 'virtualbox' provider...
    Bringing machine 'kubenode02' up with 'virtualbox' provider...
    ==> kubemaster: Importing base box 'ubuntu/bionic64'...
    ==> kubemaster: Matching MAC address for NAT networking...
    ==> kubemaster: Setting the name of the VM: kubemaster
    ==> kubemaster: Clearing any previously set network interfaces...
    ==> kubemaster: Preparing network interfaces based on configuration...
        kubemaster: Adapter 1: nat
        kubemaster: Adapter 2: hostonly
    ==> kubemaster: Forwarding ports...
        kubemaster: 22 (guest) => 2222 (host) (adapter 1)
    ==> kubemaster: Running 'pre-boot' VM customizations...
    ==> kubemaster: Booting VM...
    ==> kubemaster: Waiting for machine to boot. This may take a few minutes...
        kubemaster: SSH address: 127.0.0.1:2222
        kubemaster: SSH username: vagrant
        kubemaster: SSH auth method: private key
        kubemaster: Warning: Connection reset. Retrying...
        kubemaster: Warning: Connection aborted. Retrying...
        kubemaster: 
        kubemaster: Vagrant insecure key detected. Vagrant will automatically replace
        kubemaster: this with a newly generated keypair for better security.
        kubemaster: 
        kubemaster: Inserting generated public key within guest...
        kubemaster: Removing insecure key from the guest if it's present...
        kubemaster: Key inserted! Disconnecting and reconnecting using new SSH key...
    ==> kubemaster: Machine booted and ready!
    ==> kubemaster: Checking for guest additions in VM...
        kubemaster: The guest additions on this VM do not match the installed version of
        kubemaster: VirtualBox! In most cases this is fine, but in rare cases it can
        kubemaster: prevent things such as shared folders from working properly. If you see
        kubemaster: shared folder errors, please make sure the guest additions within the
        kubemaster: virtual machine match the version of VirtualBox you have installed on
        kubemaster: your host and reload your VM.
        kubemaster:
        kubemaster: Guest Additions Version: 5.2.42
        kubemaster: VirtualBox Version: 7.0
    ==> kubemaster: Setting hostname...
    ==> kubemaster: Configuring and enabling network interfaces...
    ==> kubemaster: Mounting shared folders...
        kubemaster: /vagrant => C:/Users/MSI BRAVO/kubernetes-install-cluster-with-kubeadm
    ==> kubenode01: Importing base box 'ubuntu/bionic64'...
    ==> kubenode01: Matching MAC address for NAT networking...
    ==> kubenode01: Setting the name of the VM: kubenode01
    ==> kubenode01: Fixed port collision for 22 => 2222. Now on port 2200.
    ==> kubenode01: Clearing any previously set network interfaces...
    ==> kubenode01: Preparing network interfaces based on configuration...
        kubenode01: Adapter 1: nat
        kubenode01: Adapter 2: hostonly
    ==> kubenode01: Forwarding ports...
        kubenode01: 22 (guest) => 2200 (host) (adapter 1)
    ==> kubenode01: Running 'pre-boot' VM customizations...
    ==> kubenode01: Booting VM...
    ==> kubenode01: Waiting for machine to boot. This may take a few minutes...
        kubenode01: SSH address: 127.0.0.1:2200
        kubenode01: SSH username: vagrant
        kubenode01: SSH auth method: private key
        kubenode01: Warning: Connection reset. Retrying...
        kubenode01: Warning: Connection aborted. Retrying...
        kubenode01: 
        kubenode01: Vagrant insecure key detected. Vagrant will automatically replace
        kubenode01: this with a newly generated keypair for better security.
        kubenode01: 
        kubenode01: Inserting generated public key within guest...
        kubenode01: Removing insecure key from the guest if it's present...
        kubenode01: Key inserted! Disconnecting and reconnecting using new SSH key...
    ==> kubenode01: Machine booted and ready!
    ==> kubenode01: Checking for guest additions in VM...
        kubenode01: The guest additions on this VM do not match the installed version of
        kubenode01: VirtualBox! In most cases this is fine, but in rare cases it can
        kubenode01: prevent things such as shared folders from working properly. If you see
        kubenode01: shared folder errors, please make sure the guest additions within the
        kubenode01: virtual machine match the version of VirtualBox you have installed on
        kubenode01: your host and reload your VM.
        kubenode01:
        kubenode01: Guest Additions Version: 5.2.42
        kubenode01: VirtualBox Version: 7.0
    ==> kubenode01: Setting hostname...
    ==> kubenode01: Configuring and enabling network interfaces...
    ==> kubenode01: Mounting shared folders...
        kubenode01: /vagrant => C:/Users/MSI BRAVO/kubernetes-install-cluster-with-kubeadm
    ==> kubenode02: Importing base box 'ubuntu/bionic64'...
    ==> kubenode02: Matching MAC address for NAT networking...
    ==> kubenode02: Setting the name of the VM: kubenode02
    ==> kubenode02: Fixed port collision for 22 => 2222. Now on port 2201.
    ==> kubenode02: Clearing any previously set network interfaces...
    ==> kubenode02: Preparing network interfaces based on configuration...
        kubenode02: Adapter 1: nat
        kubenode02: Adapter 2: hostonly
    ==> kubenode02: Forwarding ports...
        kubenode02: 22 (guest) => 2201 (host) (adapter 1)
    ==> kubenode02: Running 'pre-boot' VM customizations...
    ==> kubenode02: Booting VM...
    ==> kubenode02: Waiting for machine to boot. This may take a few minutes...
        kubenode02: SSH address: 127.0.0.1:2201
        kubenode02: SSH username: vagrant
        kubenode02: SSH auth method: private key
        kubenode02: Warning: Connection reset. Retrying...
        kubenode02: Warning: Connection aborted. Retrying...
        kubenode02: 
        kubenode02: Vagrant insecure key detected. Vagrant will automatically replace
        kubenode02: this with a newly generated keypair for better security.
        kubenode02: 
        kubenode02: Inserting generated public key within guest...
        kubenode02: Removing insecure key from the guest if it's present...
        kubenode02: Key inserted! Disconnecting and reconnecting using new SSH key...
    ==> kubenode02: Machine booted and ready!
    ==> kubenode02: Checking for guest additions in VM...
        kubenode02: The guest additions on this VM do not match the installed version of
        kubenode02: VirtualBox! In most cases this is fine, but in rare cases it can
        kubenode02: prevent things such as shared folders from working properly. If you see
        kubenode02: shared folder errors, please make sure the guest additions within the
        kubenode02: virtual machine match the version of VirtualBox you have installed on
        kubenode02: your host and reload your VM.
        kubenode02:
        kubenode02: Guest Additions Version: 5.2.42
        kubenode02: VirtualBox Version: 7.0
    ==> kubenode02: Setting hostname...
    ==> kubenode02: Configuring and enabling network interfaces...
    ==> kubenode02: Mounting shared folders...
        kubenode02: /vagrant => C:/Users/MSI BRAVO/kubernetes-install-cluster-with-kubeadm

You can verify our provisioning VM by command:

    vagrant status

It will show your virtual machines which be managed by vagrant

    Current machine states:

    kubemaster                running (virtualbox)
    kubenode01                running (virtualbox)
    kubenode02                running (virtualbox)

    This environment represents multiple VMs. The VMs are all listed
    above with their current state. For more information about a specific
    VM, run `vagrant status NAME`.

#### Issue: vagrant up times out on 'default: SSH auth method: private key' 

This issue happened when failed to boot virtual machine. By default, VirtualBox uses a TSC mode called "RealTscOffset," which adjusts the TSC value on the guest machine to compensate for any time differences between the host and guest. 
If you're using `Windows` and already have `Hyper-V`, you must disable `Hyper-V` to avoid conflict with `Virtual Box` which could lead to `vagrant up` time out. 

To disable `Hyper-V` completely, enter the following command in cmd:

    bcdedit /set hypervisorlaunchtype off

followed by turn off & turn on machine. 

>Note that `bcdedit` is short for `boot configuration data edit`, i.e. it affects what software will be loaded on the next OS boot, so it is essential that you perform a full boot from a complete power down (not a suspend and restart) in order for the changes to take effect. Leave the PC powered down for `10 seconds` before starting it again. If your PC does not offer a full shutdown from the start menu you could try running `shutdown /p` from an admin command prompt. On a laptop you may have to remove the battery.

Reinstall `Vagrant` & `Virtual Box`. If issue is still exist, you might have to reinstall `Windows OS`!

## Remote to virtual machine with Vagrant

Just run the commands: 

    vagrant ssh <hostname>

![Vagrant SSH in VSCode](../images/vagrant-ssh-vscode.png)

As you can see in the output of `vagrant up`, Vagrant had fowarded port 22 and generated keypairs for each machine without ssh configuration in `Vagrantfile`. 
For more information you can see [Vagrant Share: SSH Sharing](https://developer.hashicorp.com/vagrant/docs/share/ssh) and [Vagrantfile: config.ssh](https://developer.hashicorp.com/vagrant/docs/vagrantfile/ssh_settings).

Then you're good to go!

## Next

▶️ [Installing a container runtime (containerd) on all virtual machines](Installing-a-container-runtime.md)