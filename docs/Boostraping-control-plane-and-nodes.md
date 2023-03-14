# Boostraping control plane and nodes
<p align="left">
<img src="https://kubernetes.io/images/favicon.png" width="50" height="50">
</p>

>Note:If you have already installed kubeadm long time before, run `apt-get update && apt-get upgrade` to get the latest version of `kubeadm` & `kubelet`.

![Components of kubernetes](images/components-of-kubernetes.svg)

### Initializing your control-plane node

The `control-plane node` is the machine where the control plane components run, including `etcd` (the cluster database) and the `API Server` (which the `kubectl` command line tool communicates with).

To initialize the control-plane node, run this command in your virtual machine hostname `kubemaster`:

    kubeadm init

Some options of `kubeadm init` 
* `--apiserver-advertise-address=<string>`: The IP address the API Server will advertise it's listening on. If not set the default network interface will be used, in this tutorial it will be IP address of `kubemaster` vm. (When our cluster have more than one `control plane node`, this option will help to specify which `control plane node` listen to our `kubectl`)
* `--pod-network-cidr=<string>`: Specify range of IP addresses for the pod network. If set, the control plane will automatically allocate CIDRs for every node. By default, `kubeadm init` uses the `--pod-network-cidr=10.244.0.0/16` flag, which specifies a range of IP addresses that can be used for pod IPs. However, this default value may not be suitable for all environments. **You may need to choose a different CIDR range that is not overlap with any existing network ranges to avoid IP address conflicts**.

`kubeadm init` first runs a series of prechecks to ensure that the machine is ready to run `Kubernetes`. These prechecks expose warnings and exit on errors. `kubeadm init` then downloads and installs the cluster control plane components. This may take several minutes. After it finishes you should see: