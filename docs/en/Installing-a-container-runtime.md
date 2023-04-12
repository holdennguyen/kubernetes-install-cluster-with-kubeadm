# Installing a container runtime (containerd) on all virtual machines
<p align="left">
<img src="https://containerd.io/img/logos/icon/black/containerd-icon-black.png" width="50" >
</p>

*Performing this task on all virtual machines*

> Note: Dockershim has been removed from the Kubernetes project as of release 1.24. Read the [Dockershim Removal FAQ](https://kubernetes.io/blog/2022/02/17/dockershim-faq/) for further details.

> Dockershim is a component of Kubernetes that was used to communicate with the Docker runtime. It was introduce as a temporary solution to allow Kubernetes to use Docker as a container runtime, before Kubernetes had its own `container runtime interface (CRI)`.

You need to install a container runtime into each node in the cluster so that Pods can run there. And Kubernetes 1.26 requires that you use a runtime that conforms with the `Container Runtime Interface (CRI)`. There are several common container runtimes with Kubernetes:
* [containerd](https://containerd.io/)
* [CRI-O](https://cri-o.io/)
* [Docker Engine](https://docs.docker.com/engine/)
* [Mirantis Container Runtime](https://docs.mirantis.com/mcr/20.10/overview.html)

You can find the instructions for each type [here](https://kubernetes.io/docs/setup/production-environment/container-runtimes/). In this tutorial, we will use [**Containerd**](https://github.com/containerd/containerd/blob/main/docs/getting-started.md).

## Install and configure prerequisites

### Load kernel modules in Linux

    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF

    sudo modprobe overlay
    sudo modprobe br_netfilter

* The `modules-load.d` directory in Linux is a system directory used for configuring the kernel module loading process. It contains files with the `.conf` extension that specify the modules that should be loaded when the system boots up.

* The module `overlay` is used to provide the overlay filesystem, which is a type of filesystem that allows multiple filesystems to be layered on top of each other. This is useful in containerization technology, where each container needs its isolated filesystem.

* The module `br_netfilter` is used to enable kernel-level packet filtering and manipulation, which is useful for network traffic control and security. It is often used in conjunction with network namespaces and virtual network devices to provide network isolation and routing for containerized applications.

Verify that the `overlay`, `br_netfilter` modules are loaded by running the below instructions:

    lsmod | grep overlay
    lsmod | grep br_netfilter

### Forwarding IPv4 and letting iptables see bridged traffic

    # sysctl params required by setup, params persist across reboots
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF

    # Apply sysctl params without reboot
    sudo sysctl --system

* The `sysctl.d` directory in Linux is a system directory used for configuring kernel parameters at runtime. It contains files with the `.conf` extension that specifies the values of sysctl variables, which are kernel parameters that can be used to fine-tune the behavior of the Linux kernel.

* In a containerized environment, it is often necessary to enable (set to 1) `net.bridge.bridge-nf-call-iptables` so that traffic between containers can be filtered by the host's iptables firewall. This is important for security reasons, as it allows the host to provide an additional layer of network security for containerized applications.

Verify that the `net.bridge.bridge-nf-call-iptables`, `net.bridge.bridge-nf-call-ip6tables`, `net.ipv4.ip_forward` system variables are set to 1 in your sysctl config by running the below instruction:

    sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward

## Install containerd

The `containerd.io` packages in DEB and RPM formats are distributed by Docker (not by the containerd project). Compare to the containerd original binaries, `containerd.io` package contains runc too, but does not contain CNI plugins.

>Container Network Interface (CNI) is a standard interface for configuring networking for Linux containers. It enables a wide range of networking options, including overlay networks, load balancing, and security policies, to be used with containerized applications. In this tutorial we use CNI plugin based on Pod network add-on which will be installed later in task [Boostrapping control plane and nodes](/docs/en/Boostrapping-control-plane-and-nodes.md/##installing-a-pod-network-add-on).

Update the apt package index and install packages to allow apt to use a repository over HTTPS:

    sudo apt-get update

    sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

Add Docker’s official GPG key:

    sudo mkdir -m 0755 -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

Use the following command to set up the repository:

    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

Update the apt package index after setting up the repo:

    sudo apt-get update

Install the latest version package `containerd.io`

    sudo apt-get install containerd.io

## Cgroup drivers

On Linux, [control groups](https://docs.kernel.org/admin-guide/cgroup-v1/cgroups.html) are used to constrain resources that are allocated to processes.

Both kubelet and the underlying container runtime need to interface with control groups to enforce resource management for pods and containers and set resources such as CPU/memory requests and limits. To interface with control groups, the kubelet and the container runtime need to use a cgroup driver. **It's critical that the `kubelet` and the `container runtime` uses the same `cgroup driver` and are configured the same**.

There are two cgroup drivers available:

* **cgroupfs**
* **systemd**

Since our virtual machine use **systemd**, we will configure `kubelet` and `containerd` use **systemd** as their `cgroup driver`.


>Depending on the distribution and version, you may see the difference `cgroup driver`. To show the current `cgroup driver` type in Linux, you can check the value of the cgroup mount point by type the following command: `cat /proc/mounts | grep cgroup`

#### Configuring the `containerd` cgroup driver

To config `containerd` use the `systemd` cgroup driver, run:

    sudo vi /etc/containerd/config.toml

replace all context of `config.toml` file with the setting below: 

    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true

make sure to restart `containerd` to apply this change

    sudo systemctl restart containerd

#### Configuring the `kubelet` cgroup driver

In v1.22, if the user is not setting the `cgroupDriver` field under `KubeletConfiguration`, `kubeadm` will default it to **systemd**. 
So in this tutorial, we don't need to config `cgroup driver` for `kubelet` cause we will use `kubeadm` bootstrapping the cluster in the next few steps.
You can see [here](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#configuring-the-kubelet-cgroup-driver) for more configuring information.

## Next

▶️ [Installing kubeadm, kubelet and kubectl on all virtual machines](Installing-kubeadm-kubelet-kubectl.md/#installing-kubeadm-kubelet-and-kubectl)