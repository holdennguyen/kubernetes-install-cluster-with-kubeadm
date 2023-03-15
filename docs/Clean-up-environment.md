# Clean up environment
<p align="left">
<img src="data:image/png;base64,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" width="70">
</p>

Unsolved issue or just want to start over, this guide is for you!

## Keep virtual machine, just clean up Kubernetes cluster

#### Remove the node

Run this to gracefully evict all the running pods on the node:

    kubectl drain <node name> --delete-emptydir-data --force --ignore-daemonsets

Reset the state installed by kubeadm

    kubeadm reset

The reset process does not reset or clean up iptables rules or IPVS tables. If you wish to reset iptables, you must do so manually:

    iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X

If you want to reset the IPVS tables, you must run the following command:

    ipvsadm -C

Now remove the node:

    kubectl delete node <node name>

If you wish to start over, run `kubeadm init` or `kubeadm join` with the appropriate arguments.

#### Clean up the control plane

Performs a best effort revert of changes made to this host by `kubeadm init`

    sudo kubeadm reset --kubeconfig="$HOME/.kube/config"

`--kubeconfig=string`: Delete the kubeconfig file to use when talking to the cluster. If the flag is not set, a set of standard locations can be searched for an existing kubeconfig file. (Default: `/etc/kubernetes/admin.conf`)

The reset process does not reset or clean up iptables rules or IPVS tables. If you wish to reset iptables, you must do so manually as we do in [Remove the node](#remove-the-node).

## Dispose all virtual machines

Since we use `Vagrant` to automate provisioning the virtual machines, you can dispose all that with just a command. Run this in project directory of your host, which install `Vagrant` & `Virtual Box`:

    vagrant destroy

>If you just want to turn off the virtual machines, run `vagrant halt` instead. When you `vagrant up` again, all your work will not be remove. Read more about this command [here](https://developer.hashicorp.com/vagrant/docs/cli/halt).