# Container Runtimes

> Note: Dockershim has been removed from the Kubernetes project as of release 1.24. Read the [Dockershim Removal FAQ](https://kubernetes.io/blog/2022/02/17/dockershim-faq/) for further details.

> Dockershim is a component of Kubernetes that was used to communicate with the Docker runtime. It was introduce as a temporary solution to allow Kubernetes to use Docker as a container runtime, before Kubernetes had its own container runtime interface (CRI).

You need to install a container runtime into each node in the cluster so that Pods can run there. And Kubernetes 1.26 requires that you use a runtime that conforms with the Container Runtime Interface (CRI). There are several common container runtimes with Kubernetes:
* [containerd](https://containerd.io/)
* [CRI-O](https://cri-o.io/)
* [Docker Engine](https://docs.docker.com/engine/)
* [Mirantis Container Runtime](https://docs.mirantis.com/mcr/20.10/overview.html)

You can find the instructions for each type [here](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-versions). In this tutorial we will use `containerd`.

## Install and configure prerequistes

### Load kernel modules in Linux

    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF

    sudo modprobe overlay
    sudo modprobe br_netfilter

* The `modules-load.d` directory in Linux is a system directory used for configuring the kernel module loading process. It contains files with the `.conf` extension that specify the modules that should be loaded when the system boots up.

* The `overlay` module is used to provide the overlay filesystem, which is a type of filesystem that allows multiple filesystems to be layered on top of each other. This is useful in containerization technology, where each container needs its own isolated filesystem.

* The `br_netfilter` module is used to enable kernel-level packet filtering and manipulation, which is useful for network traffic control and security. It is often used in conjunction with network namespaces and virtual network devices to provide network isolation and routing for containerized applications.

Verify that the `overlay`, `br_netfilter` modules are loaded by running below instructions:

    lsmod | grep overlay
    lsmod | grep br_netfilter

### Fowarding IPv4 and letting iptables see bridged traffic

    # sysctl params required by setup, params persist across reboots
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF

    # Apply sysctl params without reboot
    sudo sysctl --system

* The `sysctl.d` directory in Linux is a system directory used for configuring kernel parameters at runtime. It contains files with the `.conf` extension that specify the values of sysctl variables, which are kernel parameters that can be used to fine-tune the behavior of the Linux kernel.

* In a containerized environment, it is often necessary to enable (set to 1) `net.bridge.bridge-nf-call-iptables` so that traffic between containers can be filtered by the host's iptables firewall. This is important for security reasons, as it allows the host to provide an additional layer of network security for containerized applications.

Verify that the `net.bridge.bridge-nf-call-iptables`, `net.bridge.bridge-nf-call-ip6tables`, `net.ipv4.ip_forward` system variables are set to 1 in your sysctl config by running below instruction:

    sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward

## Cgroup drivers

On Linux, [control groups](https://docs.kernel.org/admin-guide/cgroup-v1/cgroups.html) are used to constrain resources that are allocated to processes.

Both kubelet and the underlying container runtime need to interface with control groups to enforce resource management for pods and containers and set resources such as cpu/memory requests and limits. To interface with control groups, the kubelet and the container runtime need to use a cgroup driver. **It's critical that the `kubelet` and the `container runtime` uses the same `cgroup driver` and are configured the same**.

There are two cgroup drivers available:

* cgroupfs
* systemd



## Remote to virtual machine with Vagrant

