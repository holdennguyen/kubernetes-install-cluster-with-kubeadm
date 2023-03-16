# Dọn dẹp môi trường
<p align="left">
<img src="/docs/images/cleaning.png" width="70">
</p>

Sẽ có lúc bạn gặp những lỗi không biết cách giải quyết hoặc đơn giản chỉ muốn bắt đầu lại từ đầu, phần này sẽ dành cho bạn!

## Giữ lại các máy ảo, chỉ dọn dẹp Kubernetes cluster

#### Loại bỏ node

Chạy lệnh này để bỏ tất cả các `pod` đang chạy trên `node` theo đúng quy trình:

    kubectl drain <node name> --delete-emptydir-data --force --ignore-daemonsets

Reset các trạng thái được cài đặt bởi kubeadm

    kubeadm reset

Quá trình reset này sẽ không bao gồm việc reset hay dọn dẹp `iptables rules` hay `IPVS tables`. Nếu bạn muốn reset `iptables`, phải thực hiện thủ công như sau:

    iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X

Nếu muốn reset `IPVS tables`, bạn phải chạy lệnh dưới đây:

    ipvsadm -C

Bây giờ tiến hành loại bỏ `node` khỏi `cluster`:

    kubectl delete node <node name>

Nếu muốn thiết lập lại, run `kubeadm init` (thiết lập thành `control-plane`) hoặc `kubeadm join` (thiết lập thành `worker node`) với các đối số phù hợp.

#### Dọn dẹp control plane

Tiến hành quá trình đảo ngược lại tất cả các thay đổi `kubeadm init` đã thực hiện trên máy với lệnh:

    sudo kubeadm reset --kubeconfig="$HOME/.kube/config"

`--kubeconfig=string`: Xóa `kubeconfig file` được dùng để giao tiếp với `cluster`. Nếu không khai báo, một số thư mục sẽ được tìm để xóa `kubeconfig file`. (Nếu bạn thiết lập cho `non-root user` sau khi chạy `kubeadm init`, đừng quên xóa tệp thiết lập `$HOME/.kube/config`)

Tương tự như đã đề cập ở phần loại bỏ `node`, quá trình reset này sẽ không reset hay dọn dẹp `iptables rules` hay `IPVS tables`. Nếu muốn reset, bạn phải làm thủ công như khi [Loại bỏ node](#remove-the-node)

## Xóa bỏ tất cả các máy ảo

Bởi vì sử dụng `Vagrant` để tự động hóa quá trình tạo các máy ảo, bạn có thể xóa bỏ tất cả chỉ với một câu lệnh. Chạy lệnh này ở thư mục đang sử dụng trên máy tính (Nơi cài đặt `Vagrant` và `Virtual Box`):

    vagrant destroy

>Nếu bạn chỉ muốn tắt các máy ảo, thay vào đó hãy chạy lệnh `vagrant halt`. Khi đó, lúc `vagrant up` lại, tất cả công việc đã thực hiện trên máy ảo sẽ vẫn còn đó thay vì mất hết. Tìm hiểu thêm về lệnh này tại [đây](https://developer.hashicorp.com/vagrant/docs/cli/halt).