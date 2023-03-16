<h1 align="center">
<br>
  <a href="README.md"><img src="docs/images/cluster-k8s.png" alt="Cluster diagram"></a>
  <br>
    <br>
  Bootstrapping Kubernetes Cluster with kubeadm on Virtual Box
  <br>
</h1>

<h4 align="center">🦖 Detail explain step-by-step for beginner.</h4>
<p align="center">Reference to K8s documentation - section <a href="https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/" target="_blank">Bootstrapping clusters with kubeadm.</a><br>This tutorial walks you through setting up Kubernetes Cluster on a local machine using VirtualBox.<br>We will use Vagrant to automate provisioning VirtualBox's VM.</p>

<p align="center">
  <a href="https://www.vagrantup.com/" target="_blank">
    <img src="https://img.shields.io/badge/-Vagrant-1868F2?logo=vagrant&logoColor=white" alt="Vagrant">
  </a>
  <a href="https://www.virtualbox.org/" target="_blank">
    <img src="https://img.shields.io/badge/-VirtualBox-183A61?logo=VirtualBox&logoColor=white" alt="Kubernetes">
  </a>
  <a href="https://kubernetes.io/" target="_blank">
    <img src="https://img.shields.io/badge/-Kubernetes-326CE5?logo=kubernetes&logoColor=white" alt="Kubernetes">
  </a>
  <a href="https://containerd.io/" target="_blank">
    <img src="https://img.shields.io/badge/-containerd-575757?logo=containerd&logoColor=white" alt="containerd">
  </a>
</p>

<br>

<p align="center">
    <b>LANGUAGE</b>
</p>
<p align="center">
    <a href="README.md"><img src="/docs/images/us.png" width="25"></a>
    <a href="README-vi.md"><img src="/docs/images/vi.png" width="25"></a>
</p>

### Before you begin
* 🚧 Cluster diagram: 1 Control Plan & 2 Node.
* 🖥️ Machine operating system: Ubuntu 18.04 LTS (Bionic Beaver)
* ⚙️ System resource: 2 GB of RAM and 2 CPUs per machine.
* 📮 Unique hostname, MAC address, and product_uuid for every machine.
* 🧱 No firewall configuration. (Allow all traffic by default)
* 🌐 Full network connectivity between all machines in the cluster (private network, network interface enp0s8 of virtual machines).

### Step-by-step tutorial

* ▶️ [Provision VirtualBox's VM with Vagrant](docs/en/Provision-VirtualBoxVM-with-Vagrant.md)
* ▶️ [Installing a container runtime (containerd) on all virtual machines](docs/en/Installing-a-container-runtime.md)
* ▶️ [Installing kubeadm, kubelet and kubectl on all virtual machines](docs/en/Installing-kubeadm-kubelet-kubectl.md)
* ▶️ [Bootstrapping control plane and nodes](docs/en/Boostrapping-control-plane-and-nodes.md)
* ▶️ [Clean up environment](docs/en/Clean-up-environment.md)
