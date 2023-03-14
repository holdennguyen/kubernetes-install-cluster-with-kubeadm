# Vagrant

- Website: [https://www.vagrantup.com/](https://www.vagrantup.com/)
- Source: [https://github.com/hashicorp/vagrant](https://github.com/hashicorp/vagrant)
- HashiCorp Discuss: [https://discuss.hashicorp.com/c/vagrant/24](https://discuss.hashicorp.com/c/vagrant/24)

Vagrant is a tool for creating and managing virtual machine environments. It is often considered a form of Infrastructure as Code (IaC), as it allows us to define and manage our infrastructure using code. However, it is important to note that Vagrant is primarily a tool for managing virtual machine environments, and is not typically used for managing production infrastructure.

## Quick Start

For the quick-start, we'll bring up a development machine on VirtualBox because it is free and works on all major platforms.

First, download and install the [VirtualBox](https://www.virtualbox.org/wiki/Download_Old_Builds) and [Vagrant](https://www.vagrantup.com/downloads.html) on your host.

We will build our virtual environment from Vagrantfile. Clone this repo or just copy this [Vagrantfile](../Vagrantfile) to your current directory.
<script src="https://gist.github.com/holdennguyen/78ddd9c3329a5be8c713f9e1de82e640.js"></script>

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

Note: If you're using Windows and already have Hyper-V, you must [disable Hyper-V](https://learn.microsoft.com/en-us/troubleshoot/windows-client/application-management/virtualization-apps-not-work-with-hyper-v) to avoid conflict with Virtual Box.

![Vagrant up output](images/vagrant-up.png)

You can verify our provisioning VM by command:

    vagrant status

![Vagrant status](images/vagrant-status.png)

## Remote to virtual machine with Vagrant

Just run the commands: 

    vagrant ssh <hostname>

![Vagrant SSH in VSCode](images/vagrant-ssh-vscode.png)

As you can see in the output of `vagrant up`, Vagrant had fowarded port 22 and generated keypairs for each machine without ssh configuration in Vagrantfile. 
For more information you can see [Vagrant Share: SSH Sharing](https://developer.hashicorp.com/vagrant/docs/share/ssh) and [Vagrantfile: config.ssh](https://developer.hashicorp.com/vagrant/docs/vagrantfile/ssh_settings).

Then you're good to go!
