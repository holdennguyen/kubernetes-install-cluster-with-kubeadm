# Boostraping control plane and nodes
<p align="left">
<img src="https://kubernetes.io/images/favicon.png" width="50" height="50">
</p>

>Note:If you have already installed kubeadm long time before, run `apt-get update && apt-get upgrade` to get the latest version of `kubeadm` & `kubelet`.

![Components of kubernetes](images/components-of-kubernetes.svg)

### Initializing your control-plane node

The `control-plane node` is the machine where the control plane components run, including `etcd` (the cluster database) and the `API Server` (which the `kubectl` command line tool communicates with).

To initialize the control-plane node, run this command in your virtual machine hostname `kubemaster`:

    kubeadm init --apiserver-advertise-address=192.168.56.2 --pod-network-cidr=10.244.0.0/16


