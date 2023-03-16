# Container Runtimes
<p align="left">
<img src="https://containerd.io/img/logos/icon/black/containerd-icon-black.png" width="50" >
</p>

*Thực hiện công việc ở bước này trên tất cả các máy ảo*

> Ghi chú: `Dockershim` đã bị bỏ khỏi dự án `Kubernetes` ở bản `1.24`. Đọc [Dockershim Removal FAQ](https://kubernetes.io/blog/2022/02/17/dockershim-faq/) để biết thêm thông tin.

> `Dockershim` là một thành phẩn của `Kubernetes` được sử dụng để giao tiếp với `Docker runtime`. Nó được giới thiệu như một giải pháp tạm thời cho phép `Kubernetes` sử dụng `Docker` như một `container runtime` trước khi `Kubernetes` có `container runtime interface (CRI)` của riêng họ.

Bạn cần phải cài đặt một `container runtime` trên mỗi node trong cụm K8s (Kubernetes) để các `Pods` có thể chạy ở đó. Và phiên bản K8s 1.26 yêu cầu phải sử dụng một `container runtime` tương thích với `Container Runtime Interface (CRI)` của K8s. Đây là một số `container runtime` phổ biến với Kubernetes:
* [containerd](https://containerd.io/)
* [CRI-O](https://cri-o.io/)
* [Docker Engine](https://docs.docker.com/engine/)
* [Mirantis Container Runtime](https://docs.mirantis.com/mcr/20.10/overview.html)

Bạn có thể xem hướng dẫn cài đặt cho các loại trên tại [đây](https://kubernetes.io/docs/setup/production-environment/container-runtimes/). Trong hướng dẫn này, chúng ta sẽ sử dụng [**Containerd**](https://github.com/containerd/containerd/blob/main/docs/getting-started.md).

## Cài đặt và thiết lập các yêu cầu cần chuẩn bị trước

### Tải các module của nhân Linux

    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF

    sudo modprobe overlay
    sudo modprobe br_netfilter

* `modules-load.d` trong Linux là thư mục hệ thống sử dụng để thiết lập các module của kernel được tải lên tiến trình. Nó bao gồm các tệp đuôi `.conf` chỉ định các module được tải khi hệ thống khởi động.

* Module `overlay` được sử dụng để cung cấp overlay filesystem, là một kiểu filesystem cho phép nhiều filesystem xếp chồng lên nhau. Nó cực kỳ hữu dụng trong công nghệ containerization, nơi các container cần những filesystem cô lập của riêng nó.

* Module `br_netfilter` được sử dụng để bật tính năng lọc và thao tác gói tin ở kernel-level, hữu dụng đối với việc kiểm soát lưu lượng mạng và bảo mật. Nó thường được dùng cùng với các mạng của namespace và các thiết bị mạng ảo để cung cấp việc cô lập cũng như định tuyến cho ứng dụng containerized.  

Để kiểm tra module `overlay` và `br_netfilter` đã được load, chạy lệnh dưới đây:

    lsmod | grep overlay
    lsmod | grep br_netfilter

### Chuyển tiếp IPv4 và cho phép iptables nhận diện brigded traffic

    #  thiết lập các tham số sysctl, luôn tồn tại dù khởi động lại
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF

    # Áp dụng các tham số sysctl mà không cần khởi động lại
    sudo sysctl --system

* `sysctl.d` là thư mục hệ thống trong Linux, sử dụng để thiết lập các tham số cho kernel trong runtime. Nó chứa các tệp có đuôi `.conf` quy định các giá trị của biến sysctl, đó là các tham số của kernel có thể được sử dụng để tinh chỉnh hành vi của Linux kernel.

* Trong môi trường containerized, thông thường cần phải bật (đặt giá trị 1) `net.bridge.bridge-nf-call-iptables` để traffic giữa các container có thể lọc bởi iptables firewall của máy chủ. Điều này rất quan trọng vì lý do bảo mật, nó cho phép máy chủ cung cấp các lớp bảo mật mạng cho ứng dụng containerized.

Để kiểm tra `net.bridge.bridge-nf-call-iptables`, `net.bridge.bridge-nf-call-ip6tables`, `net.ipv4.ip_forward` đã được bật trong thiết lập sysctl hay chưa, chạy lệnh:

    sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward

## Cài đặt containerd

Gói `containerd.io` ở định dạng DEB và RPM được phân phối bởi `Docker` (không phải bởi dự án `containerd`). So sánh với các tệp nhị phân gốc của `containerd`, gói `containerd.io` cũng bao gồm `runc`, nhưng lại không có `CNI plugins`.

>`Container Network Interface (CNI)` là giao diện tiêu chuẩn để cấu hình mạng cho các `container` trên `Linux`. Nó cho phép một loạt các tùy chọn kết nối mạng bao gồm `overlay network`, `load balancing` và các chính sách bảo mật sử dụng với ứng dụng containerized. Trong hướng dẫn này, chúng ta sẽ sử dụng `CNI plugin` với `Pod network add-on` sẽ được cài đặt ở bước sau trong mục [Khởi tạo control plane và nodes](/docs/vi/Boostrapping-control-plane-and-nodes.md/##installing-a-pod-network-add-on).

Cập nhật apt package index và cài đặt packages cho phép apt sử dụng repository qua HTTPS:

    sudo apt-get update

    sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

Thêm GPG key chính thức của Docker:

    sudo mkdir -m 0755 -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

Sử dụng lệnh sau để cài đặt repository:

    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

Cập nhật lại apt package index sau khi cài đặt repo:

    sudo apt-get update

Cài đặt phiên bản mới nhất của gói `containerd.io`

    sudo apt-get install containerd.io

## Cgroup drivers

Trong Linux, các [control group](https://docs.kernel.org/admin-guide/cgroup-v1/cgroups.html) được sử dụng để giới hạn tài nguyên phân bổ cho các tiến trình.

Các `kubelet` và `container runtime` chạy dưới nó đều cần `control groups` để thực hiện việc quản lý tài nguyên cho các `pod` và `container` như yêu cầu hay giới hạn về cpu/memory. Để giao tiếp với các `control group`, `kubelet` và `container runtime` cần sử dụng `cgroup driver`. **Một điều cực kỳ quan trọng đó là `kubelet` và `container runtime` cần phải sử dụng cùng một loại `cgroup driver` với thiết lập giống nhau**.

Có hai loại `cgroup drivers` hỗ trợ đó là:

* **cgroupfs**
* **systemd**

Bởi vì các máy ảo đã dựng của chúng ta sử dụng **systemd**, vậy nên ta sẽ thiết lập `kubelet` và `containerd` dùng **systemd** làm `cgroup driver`.


>Tùy thuộc vào bản phân phối và phiên bản của Linux, bạn sẽ thấy loại `cgroup driver` khác nhau. Để xem loại `cgroup driver` hiện tại trên Linux, bạn có thể kiểm tra giá trị của `cgroup mount point` bằng cách nhập lệnh: `cat /proc/mounts | grep cgroup`

#### Thiết lập `cgroup driver` cho `containerd`

Để thiết lập cho `containerd` dùng `cgroup driver` là `systemd`, chạy:

    sudo vi /etc/containerd/config.toml

thay thế toàn bộ nội dung trong tệp `config.toml` với nội dung cài đặt sau: 

    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true

nhớ khởi động lại `containerd` để áp dụng thay đổi

    sudo systemctl restart containerd

#### Thiết lập `cgroup driver` cho `kubelet`

Trong phiên bản 1.22, nếu người dùng không cài đặt trường `cgroupDriver` trong `KubeletConfiguration`, `kubeadm` sẽ mặc định nó là **systemd**. 
Chúng ta không cần làm gì để thiết lập `cgroup driver` cho `kubelet` trong hướng dẫn này vì sẽ dùng `kubeadm` để khởi tạo cụm K8s trong các bước tiếp theo.
Bạn có thể xem tại [đây](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#configuring-the-kubelet-cgroup-driver) để biết thêm thông tin về cách thiết lập.

## Tiếp theo

▶️ [Cài đặt kubeadm, kubelet và kubectl trên tất cả các máy ảo](Installing-kubeadm-kubelet-kubectl.md/#kubernetes-cluster-with-containerd)