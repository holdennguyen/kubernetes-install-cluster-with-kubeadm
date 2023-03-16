# Kubeadm
<p align="left">
<img src="https://d33wubrfki0l68.cloudfront.net/e4a8ddb49f07de8b2c2dbbfc7c9bedcfe0816701/600b1/images/kubeadm-stacked-color.png" width="50">
</p>

*Thực hiện công việc ở bước này trên tất cả các máy ảo*

`Kubeadm` là một command-line tool dùng để khởi tạo một `Kubernetes cluster`. Nó là một bản phân phối chính thức của `Kubernetes` và được thiết kế để đơn giản hóa quá trình thiết lập `Kubernetes cluster`. `Kubeadm` tự động hóa nhiều tác vụ liên quan đến thiết lập `cluster` chẳng hạn như cấu hình `control plane components`, tạo `TLS certificates`, và thiết lập `Kubernetes networking`.

>Một trong những nội dung chính được đề cập trong kỳ thi `Certified Kubernetes Administrator (CKA)` là thiết lập `cluster`, bao gồm việc sử dụng các công cụ như kubeadm để khởi tạo một `Kubernetes cluster` mới.

## Tắt swap space:

Bạn phải tắt tính năng `swap` để `kubelet` hoạt động bình thường. Xem thêm thảo luận về điều này trong issue: https://github.com/kubernetes/kubernetes/issues/53533

`Kubelet`, là `node agent` chính chạy trên `worker node`, giả sử mỗi `node` có một lượng bộ nhớ khả dụng cố định. Nếu `node` bắt đầu tiến hành [swap](https://web.mit.edu/rhel-doc/5/RHEL-5-manual/Deployment_Guide-en-US/ch-swapspace.html), `kubelet` có thể bị delay hoặc các vấn đề khác ảnh hưởng đến tính `stability` và `reliability` của `Kubernetes cluster`. Chính vì vậy, `Kubernetes` khuyên `swap` nên được **disabled** trên mỗi `node` trong `cluster`.

Để tắt `swap` trên máy Linux, sử dụng:

    # Đầu tiên là tắt swap
    sudo swapoff -a

    # Sau đó tắt swap mỗi khi khởi động trong /etc/fstab
    sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

## Cài đặt kubeadm, kubelet và kubectl :

* `kubelet`: component chạy trên tất cả các máy trong `cluster` và thực hiện những việc như khởi động các `pod` và `container`.

* `kubectl`: command line tool dùng để nói chuyện với `cluster`.

* `kubeadm`: công cụ cài đặt các component còn lại của `kubernetes cluster`.

`kubeadm` sẽ không cài đặt `kubelet` hay `kubectl` cho bạn, vì vậy hãy đảm bảo chúng sử dụng các phiên bản phù hợp với các `component` khác trong `Kubernetes control plane` mà `kubeadm` cài cho bạn.

>Cảnh báo: Hướng dẫn này sẽ loại bỏ các `Kubernetes packages` ra khỏi mọi tiến trình system upgrade. Do `kubeadm` và `Kubernetes` cần được đặc biệt chú ý mỗi khi upgrade.

Để biết thêm thông tin về việc các phiên bản lệch nhau được hỗ trợ hãy xem:

* Kubernetes [version and version-skew policy](https://kubernetes.io/releases/version-skew-policy/)
* Kubeadm-specific [version skew policy](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#version-skew-policy)

##### Cập nhật `apt package index` và cài các `package` cần thiết để sử dụng trong `Kubernetes apt repository`:

    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl

##### Tải `Google Cloud public signing key`:

    sudo mkdir -m 0755 -p /etc/apt/keyrings
    sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

##### Thêm `Kubernetes apt repository`:

    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

##### Cập nhật lại `apt package index`, cài đặt phiên bản mới nhất của `kubelet`, `kubeadm` và `kubectl`, ghim phiên bản hiện tại tránh việc tự động cập nhật:

    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl

`kubelet` sẽ tự động khởi động lại mỗi giây vì ở trạng thái crashloop, đợi `kubeadm` đưa ra yêu cầu cần thực hiện.

>Ghi chú: 🔐 `Client certificates` được tạo bởi `kubeadm` sẽ bị hết hạn sau `1 year`. Đọc thêm ở [đây](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/) để biết thêm về cách tùy chỉnh và làm mới `certificates`.

## Tiếp theo

▶️ [Khởi động control plane và nodes](Boostrapping-control-plane-and-nodes.md/#boostraping-control-plane-and-nodes)