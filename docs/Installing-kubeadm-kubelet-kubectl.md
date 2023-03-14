# Kubernetes Cluster with Containerd
<p align="left">
<img src="https://d33wubrfki0l68.cloudfront.net/e4a8ddb49f07de8b2c2dbbfc7c9bedcfe0816701/600b1/images/kubeadm-stacked-color.png" width="50" height="50">
<img src="https://kubernetes.io/images/favicon.png" width="50" height="50">
</p>

`Kubeadm` is a command-line tool for bootstrapping a `Kubernetes cluster`. It is part of the official `Kubernetes distribution` and is designed to simplify the process of setting up a new `Kubernetes cluster`. `Kubeadm` automates many of the tasks involved in setting up a cluster, such as configuring the `control plane components`, generating `TLS certificates`, and setting up the `Kubernetes networking`.

>One of the key areas covered in the `Certified Kubernetes Administrator (CKA)` exam is cluster setup, which includes using tools like Kubeadm to bootstrap a new Kubernetes cluster.

### Disable swap:

You must disable `swap` in order for the `kubelet` to work properly. The discussion about this is in this issue https://github.com/kubernetes/kubernetes/issues/53533

To disable `swap` on Linux Machines:

    # First diasbale swap
    sudo swapoff -a

    # And then to disable swap on startup in /etc/fstab
    sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

### Installing kubeadm, kubelet and kubectl :

![Components of kubernetes](images/components-of-kubernetes.svg)

You will install these packages on all of your machines:

* `kubelet`: the component that runs on all of the machines in your cluster and does things like starting pods and containers.

* `kubectl`: the command line util to talk to your cluster.

* `kubeadm`: the command to bootstrap the cluster. (install the rest components for you)

`kubeadm` will not install or manage `kubelet` or `kubectl` for you, so you will need to ensure they match the version of the `Kubernetes control plane` you want `kubeadm` to install for you.

>Warning: These instructions exclude all Kubernetes packages from any system upgrades. This is because kubeadm and Kubernetes require special attention to upgrade.

For more information on version skews, see:

* Kubernetes [version and version-skew policy](https://kubernetes.io/releases/version-skew-policy/)
* Kubeadm-specific [version skew policy](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#version-skew-policy)

##### Update the `apt` package index and install packages needed to use the Kubernetes `apt` repository:

    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl

##### Download the Google Cloud public signing key:

    sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

##### Add the Kubernetes apt repository:

    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

##### Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:

    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl

The kubelet is now restarting every few seconds, as it waits in a crashloop for kubeadm to tell it what to do.