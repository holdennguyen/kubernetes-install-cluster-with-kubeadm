# Vagrant

- Website: [https://www.vagrantup.com/](https://www.vagrantup.com/)
- Source: [https://github.com/hashicorp/vagrant](https://github.com/hashicorp/vagrant)
- HashiCorp Discuss: [https://discuss.hashicorp.com/c/vagrant/24](https://discuss.hashicorp.com/c/vagrant/24)

Vagrant is a tool for creating and managing virtual machine environments. It is often considered a form of Infrastructure as Code (IaC), as it allows us to define and manage our infrastructure using code. However, it is important to note that Vagrant is primarily a tool for managing virtual machine environments, and is not typically used for managing production infrastructure.

## Quick Start

For the quick-start, we'll bring up a development machine on VirtualBox because it is free and works on all major platforms.

First, download and install the [VirtualBox](https://www.virtualbox.org/wiki/Download_Old_Builds) and [Vagrant](https://www.vagrantup.com/downloads.html) on your host.

We will build our virtual environment from Vagrantfile. Clone this repo or just copy this [Vagrantfile](../Vagrantfile) to your current directory.

<iframe
  src="https://carbon.now.sh/embed?bg=rgba%28171%2C+184%2C+195%2C+1%29&t=vscode&wt=sharp&l=ruby&width=800&ds=false&dsyoff=10px&dsblur=68px&wc=false&wa=false&pv=0px&ph=0px&ln=false&fl=1&fm=Hack&fs=14px&lh=175%25&si=false&es=2x&wm=false&code=%2523%2520-*-%2520mode%253A%2520ruby%2520-*-%250A%2523%2520vi%253Aset%2520ft%253Druby%2520sw%253D2%2520ts%253D2%2520sts%253D2%253A%250A%250A%2523%2520Define%2520the%2520number%2520of%2520control%2520plane%2520%28MASTER_NODE%29%2520and%2520node%2520%28WORKER_NODE%29%250ANUM_MASTER_NODE%2520%253D%25201%250ANUM_WORKER_NODE%2520%253D%25202%250A%250AIP_NW%2520%253D%2520%2522192.168.56.%2522%250AMASTER_IP_START%2520%253D%25201%250ANODE_IP_START%2520%253D%25202%250A%250A%2523%2520All%2520Vagrant%2520configuration%2520is%2520done%2520below.%2520The%2520%25222%2522%2520in%2520Vagrant.configure%250A%2523%2520configures%2520the%2520configuration%2520version%2520%28we%2520support%2520older%2520styles%2520for%250A%2523%2520backwards%2520compatibility%29.%2520Please%2520don%27t%2520change%2520it%2520unless%2520you%2520know%2520what%250A%2523%2520you%27re%2520doing.%250AVagrant.configure%28%25222%2522%29%2520do%2520%257Cconfig%257C%250A%2520%2520%2523%2520The%2520most%2520common%2520configuration%2520options%2520are%2520documented%2520and%2520commented%2520below.%250A%2520%2520%2523%2520For%2520a%2520complete%2520reference%252C%2520please%2520see%2520the%2520online%2520documentation%2520at%250A%2520%2520%2523%2520https%253A%252F%252Fdocs.vagrantup.com.%250A%250A%2520%2520%2523%2520Every%2520Vagrant%2520development%2520environment%2520requires%2520a%2520box.%2520You%2520can%2520search%2520for%250A%2520%2520%2523%2520boxes%2520at%2520https%253A%252F%252Fvagrantcloud.com%252Fsearch.%250A%2520%2520%2523%2520Here%2520are%2520some%2520key%2520details%2520about%2520the%2520%2522ubuntu%252Fbionic64%2522%2520Vagrant%2520box%253A%250A%2520%2520%2520%2520%2523%2520Operating%2520System%253A%2520Ubuntu%252018.04%2520LTS%2520%28Bionic%2520Beaver%29%250A%2520%2520%2520%2520%2520%2520%2520%2520%2523%2520Ubuntu%252018.04%2520LTS%2520will%2520receive%2520security%2520updates%2520and%2520bug%2520fixes%2520%250A%2520%2520%2520%2520%2520%2520%2520%2520%2523%2520from%2520Canonical%252C%2520the%2520company%2520behind%2520Ubuntu%252C%2520until%2520April%25202023%2520%250A%2520%2520%2520%2520%2520%2520%2520%2520%2523%2520for%2520desktop%2520and%2520server%2520versions%252C%2520and%2520until%2520April%25202028%2520for%2520%250A%2520%2520%2520%2520%2520%2520%2520%2520%2523%2520server%2520versions%2520with%2520Extended%2520Security%2520Maintenance%2520%28ESM%29%2520enabled.%250A%2520%2520%2520%2520%2523%2520Architecture%253A%2520x86_64%2520%2864-bit%29%250A%2520%2520%2520%2520%2523%2520Disk%2520Size%253A%252010%2520GB%250A%2520%2520%2520%2520%2523%2520RAM%253A%25202%2520GB%250A%2520%2520%2520%2520%2523%2520CPUs%253A%25202%250A%2520%2520%2520%2520%2523%2520Desktop%2520Environment%253A%2520None%2520%28headless%29%250A%2520%2520%2520%2520%2523%2520Provider%253A%2520VirtualBox%250A%2520%2520config.vm.box%2520%253D%2520%2522ubuntu%252Fbionic64%2522%250A%250A%2520%2520%2523%2520Disable%2520automatic%2520box%2520update%2520checking.%2520If%2520you%2520disable%2520this%252C%2520then%250A%2520%2520%2523%2520boxes%2520will%2520only%2520be%2520checked%2520for%2520updates%2520when%2520the%2520user%2520runs%250A%2520%2520%2523%2520%2560vagrant%2520box%2520outdated%2560.%2520This%2520is%2520not%2520recommended.%250A%2520%2520config.vm.box_check_update%2520%253D%2520false%250A%250A%2520%2520%2523%2520View%2520the%2520documentation%2520for%2520the%2520VirtualBox%2520for%2520more%250A%2520%2520%2523%2520information%2520on%2520available%2520options.%250A%2520%2520%2523%2520https%253A%252F%252Fdeveloper.hashicorp.com%252Fvagrant%252Fdocs%252Fproviders%252Fvirtualbox%252Fconfiguration%250A%250A%2520%2520%2523%2520Provision%2520Control%2520Plane%250A%2520%2520%281..NUM_MASTER_NODE%29.each%2520do%2520%257Ci%257C%250A%2520%2520%2520%2520%2520%2520config.vm.define%2520%2522controlplane%2522%2520do%2520%257Cnode%257C%250A%2520%2520%2520%2520%2520%2520%2520%2520node.vm.provider%2520%2522virtualbox%2522%2520do%2520%257Cvb%257C%250A%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520vb.name%2520%253D%2520%2522controlpla"
  style="width: 1024px; height: 473px; border:0; transform: scale(0.75); overflow:hidden;"
  sandbox="allow-scripts allow-same-origin">
</iframe>

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