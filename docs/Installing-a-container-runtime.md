# Container Runtimes

> Note: Dockershim has been removed from the Kubernetes project as of release 1.24. Read the [Dockershim Removal FAQ](https://kubernetes.io/blog/2022/02/17/dockershim-faq/) for further details.

> Dockershim is a component of Kubernetes that was used to communicate with the Docker runtime. It was introduce as a temporary solution to allow Kubernetes to use Docker as a container runtime, before Kubernetes had its own container runtime interface (CRI).

You need to install a container runtime into each node in the cluster so that Pods can run there. And Kubernetes 1.26 requires that you use a runtime that conforms with the Container Runtime Interface (CRI). There are several common container runtimes with Kubernetes:
* [containerd](https://containerd.io/)
* [CRI-O](https://cri-o.io/)
* [Docker Engine](https://docs.docker.com/engine/)
* [Mirantis Container Runtime](https://docs.mirantis.com/mcr/20.10/overview.html)

You can find the instructions for each type [here](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-versions). In this tutorial we will use <span><a href="https://kubernetes.io/"><img src="https://img.shields.io/badge/-containerd-575757?logo=containerd&logoColor=white" alt="containerd"></a></span>.

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

## Install containerd

>In this tutorial, I will choose the latest version when writing this doc. You can go to the release links and select the difference version as you want.

#### Getting the official binaries
Download the `containerd-<VERSION>-<OS>-<ARCH>.tar.gz` archive from https://github.com/containerd/containerd/releases , verify its sha256sum, and extract it under `/usr/local`.

    wget -c https://github.com/containerd/containerd/releases/download/v1.7.0/containerd-1.7.0-linux-amd64.tar.gz
    wget -c https://github.com/containerd/containerd/releases/download/v1.7.0/containerd-1.7.0-linux-amd64.tar.gz.sha256sum

    # verify the integrity of downloaded file
    sha256sum containerd-1.7.0-linux-amd64.tar.gz

And extract it under `/usr/local`:

    sudo tar Cxzvf /usr/local containerd-1.7.0-linux-amd64.tar.gz

The `containerd` binary is built dynamically for glibc-based Linux distributions such as Ubuntu and Rocky Linux. This binary may not work on musl-based distributions such as Alpine Linux. Users of such distributions may have to install containerd from the source or a third party package.

#### Installing runc

Download the `runc.<ARCH>` binary from https://github.com/opencontainers/runc/releases , verify its sha256sum:

    wget -c https://github.com/opencontainers/runc/releases/download/v1.1.4/runc.amd64
    wget -c https://github.com/opencontainers/runc/releases/download/v1.1.4/runc.amd64.asc

    # verify the integrity of downloaded file
    sha256sum runc.amd64

and install it as `/usr/local/sbin/runc`:

    sudo install -m 755 runc.amd64 /usr/local/sbin/runc

The binary is built statically and should work on any Linux distribution.

#### Installing CNI plugins

Download the `cni-plugins-<OS>-<ARCH>-<VERSION>.tgz` archive from https://github.com/containernetworking/plugins/releases , verify its sha256sum:

    wget -c https://github.com/containernetworking/plugins/releases/download/v1.2.0/cni-plugins-linux-amd64-v1.2.0.tgz
    wget -c https://github.com/containernetworking/plugins/releases/download/v1.2.0/cni-plugins-linux-amd64-v1.2.0.tgz.sha256

    # verify the integrity of downloaded file
    sha256sum cni-plugins-linux-amd64-v1.2.0.tgz.sha256

and extract it under `/opt/cni/bin`:

    sudo mkdir -p /opt/cni/bin
    sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.2.0.tgz

#### Alternative options from `apt-get` or `dnf`

The `containerd.io` packages in DEB and RPM formats are distributed by Docker (not by the containerd project). See the [Docker documentation](https://docs.docker.com/desktop/install/linux-install/) for how to set up apt-get or dnf to install `containerd.io` packages.

**The `containerd.io` package contains runc too, but does not contain CNI plugins.**

>You need CNI (Container Network Interface) plugins for Kubernetes.
CNI is a standard interface for configuring networking for Linux containers. It enables a wide range of networking options, including overlay networks, load balancing, and security policies, to be used with containerized applications. Kubernetes uses CNI plugins to provide network connectivity between the pods running on the cluster. 

## Cgroup drivers

On Linux, [control groups](https://docs.kernel.org/admin-guide/cgroup-v1/cgroups.html) are used to constrain resources that are allocated to processes.

Both kubelet and the underlying container runtime need to interface with control groups to enforce resource management for pods and containers and set resources such as cpu/memory requests and limits. To interface with control groups, the kubelet and the container runtime need to use a cgroup driver. **It's critical that the `kubelet` and the `container runtime` uses the same `cgroup driver` and are configured the same**.

There are two cgroup drivers available:

* **cgroupfs**
* **systemd**

Since our virtual machine use **systemd**, we will configure `kubelet` and `container runtime` use **systemd** as their `cgroup driver`.


>Depending on the distribution and version, you may see the difference `cgroup driver`. To show the current `cgroup driver` type in Linux, you can check the value of the cgroup mount point by type the following command: `cat /proc/mounts | grep cgroup`

#### Configuring the `containerd` cgroup driver

To start `containerd` via **systemd**, download the `containerd.service` unit file from https://raw.githubusercontent.com/containerd/containerd/main/containerd.service into `/usr/local/lib/systemd/system/`

    sudo wget -P /usr/local/lib/systemd/system/ https://raw.githubusercontent.com/containerd/containerd/main/containerd.service

and run the following commands:

    systemctl daemon-reload
    systemctl enable --now containerd

#### Configuring the `kubelet` cgroup driver

In v1.22, if the user is not setting the `cgroupDriver` field under `KubeletConfiguration`, `kubeadm` will default it to **systemd**. 
So in this tutorial we don't need to config `cgroup driver` for `kubelet` cause we will use `kubeadm` to bootstraping cluster tool in the next few steps.
You can see [here](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#configuring-the-kubelet-cgroup-driver) for more configuring information.
