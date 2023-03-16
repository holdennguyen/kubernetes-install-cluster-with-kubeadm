# Vagrant
<p align="left">
<img src="/docs/images/vagrant-logo.png" width="50">
</p>


`Vagrant` là một công cụ để tạo và quản lý môi trường máy ảo. Nó thường được coi là một dạng `Infrastructure as Code (IaC)`, cho phép chúng ta khởi tạo và quản lý cơ sở hạ tầng của mình bằng code thay vì chọn bằng tay trên console. Vagrant chủ yếu đóng vai trò là công cụ sử dụng cho môi trường máy ảo và thường sẽ không dùng để quản lý cơ sở hạ tầng Production.

## Bắt đầu

Chúng ta sẽ tạo các máy ảo với `Virtual Box` bởi vì nó miễn phí và hoạt động ổn trên tất cả các hệ điều hành.

Đầu tiên, tải và cài đặt [VirtualBox](https://www.virtualbox.org/wiki/Download_Old_Builds), [Vagrant](https://www.vagrantup.com/downloads.html) trên máy tính của bạn.

Chúng ta sẽ xây dựng các máy ảo theo khai báo trong `Vagrantfile`. Dùng Git clone repo này hoặc đơn giản copy [Vagrantfile](../../Vagrantfile) này vào thư mục của bạn. (Tham khảo: [kodekloudhub](https://github.com/kodekloudhub/certified-kubernetes-administrator-course))

    # -*- mode: ruby -*-
    # vi:set ft=ruby sw=2 ts=2 sts=2:

    # Xác định số lượng máy control plane (MASTER_NODE) và máy node (WORKER_NODE)
    NUM_MASTER_NODE = 1
    NUM_WORKER_NODE = 2

    IP_NW = "192.168.56."
    MASTER_IP_START = 1
    NODE_IP_START = 2

    # Tất cả thiết lập Vagrant được khai báo dưới đây. Số "2" trong Vagrant.configure
    # là thiết lập phiên bản sử dụng
    # Đừng thay đổi trừ khi bạn biết mình đang làm gì

    Vagrant.configure("2") do |config|

    # Để tham khảo thêm, xem tài liệu tại
    # https://docs.vagrantup.com.

    # Tất cả môi trường mà Vagrant xây dựng đều cần một box. Bạn có thể tìm các
    # box tại https://vagrantcloud.com/search.
    # Đây là một số thông tin chi tiết về vagrant box "ubuntu/bionic64":
        # Hệ điều hành: Ubuntu 18.04 LTS (Bionic Beaver)
            # Ubuntu 18.04 LTS sẽ được cập nhật bảo mật và sửa lỗi 
            # từ Canonical, công ty đứng sau Ubuntu, cho đến tháng 4 năm 2023 
            # đối với bản desktop và server, và đến tháng 4 năm 2028 đối 
            # với bản server có Extended Security Maintenance (ESM).
        # Kiến trúc: x86_64 (64-bit)
        # Dung lượng bộ nhớ: 10 GB
        # RAM: 2 GB
        # CPUs: 2
        # Giao diện đồ họa: None (headless)
        # Người tạo: VirtualBox

    config.vm.box = "ubuntu/bionic64"

    # Tắt tính năng tự động cập nhật của box. Nếu tắt,
    # boxes sẽ chỉ kiểm tra cập nhật khi người dùng chạy
    # `vagrant box outdated`. Không khuyến khích.

    config.vm.box_check_update = false

    # Xem thêm tài liệu của Virtual box tại
    # https://developer.hashicorp.com/vagrant/docs/providers/virtualbox/configuration

    # Khởi tạo Control Plane
    (1..NUM_MASTER_NODE).each do |i|
        config.vm.define "kubemaster" do |node|
            node.vm.provider "virtualbox" do |vb|
                vb.name = "kubemaster"
                vb.memory = 2048
                vb.cpus = 2
            end
            node.vm.hostname = "kubemaster"
            node.vm.network :private_network, ip: IP_NW + "#{MASTER_IP_START + i}"
        end
    end


    # Khởi tạo Nodes
    (1..NUM_WORKER_NODE).each do |i|
        config.vm.define "kubenode0#{i}" do |node|
            node.vm.provider "virtualbox" do |vb|
                vb.name = "kubenode0#{i}"
                vb.memory = 2048
                vb.cpus = 2
            end
            node.vm.hostname = "kubenode0#{i}"
            node.vm.network :private_network, ip: IP_NW + "#{NODE_IP_START + i}"
        end
    end
    end

Trong `Vagrantfile` này, chúng ta đơn giản chỉ khai báo: 
- Số lượng máy ảo: `NUM_MASTER_NODE`, `NUM_WORKER_NODE`
- Địa chỉ IP: `IP_NW`, `MASTER_IP_START`, `NODE_IP_START`
- Kết nối mạng nội bộ: `node.vm.network`
- hostname riêng biệt cho mỗi máy ảo: `node.vm.hostname`
- Hệ điều hành: `config.vm.box`
- Tài nguyên hệ thống: `vb.memory`, `vb.cpus`

Cú pháp trong `Vagrantfile` là [Ruby](https://www.ruby-lang.org/en/), nhưng để viết hay chỉnh sửa bạn không cần phải hiểu về `ngôn ngữ lập trình Ruby`. Xem thêm [đây](https://developer.hashicorp.com/vagrant/docs/vagrantfile) để biết thêm thông tin về `cú pháp trong Vagrantfile`.

## Bắt đầu khởi tạo

Chạy câu lệnh:

    vagrant up

Output sẽ tương tự như sau:

    Bringing machine 'kubemaster' up with 'virtualbox' provider...
    Bringing machine 'kubenode01' up with 'virtualbox' provider...
    Bringing machine 'kubenode02' up with 'virtualbox' provider...
    ==> kubemaster: Importing base box 'ubuntu/bionic64'...
    ==> kubemaster: Matching MAC address for NAT networking...
    ==> kubemaster: Setting the name of the VM: kubemaster
    ==> kubemaster: Clearing any previously set network interfaces...
    ==> kubemaster: Preparing network interfaces based on configuration...
        kubemaster: Adapter 1: nat
        kubemaster: Adapter 2: hostonly
    ==> kubemaster: Forwarding ports...
        kubemaster: 22 (guest) => 2222 (host) (adapter 1)
    ==> kubemaster: Running 'pre-boot' VM customizations...
    ==> kubemaster: Booting VM...
    ==> kubemaster: Waiting for machine to boot. This may take a few minutes...
        kubemaster: SSH address: 127.0.0.1:2222
        kubemaster: SSH username: vagrant
        kubemaster: SSH auth method: private key
        kubemaster: Warning: Connection reset. Retrying...
        kubemaster: Warning: Connection aborted. Retrying...
        kubemaster: 
        kubemaster: Vagrant insecure key detected. Vagrant will automatically replace
        kubemaster: this with a newly generated keypair for better security.
        kubemaster: 
        kubemaster: Inserting generated public key within guest...
        kubemaster: Removing insecure key from the guest if it's present...
        kubemaster: Key inserted! Disconnecting and reconnecting using new SSH key...
    ==> kubemaster: Machine booted and ready!
    ==> kubemaster: Checking for guest additions in VM...
        kubemaster: The guest additions on this VM do not match the installed version of
        kubemaster: VirtualBox! In most cases this is fine, but in rare cases it can
        kubemaster: prevent things such as shared folders from working properly. If you see
        kubemaster: shared folder errors, please make sure the guest additions within the
        kubemaster: virtual machine match the version of VirtualBox you have installed on
        kubemaster: your host and reload your VM.
        kubemaster:
        kubemaster: Guest Additions Version: 5.2.42
        kubemaster: VirtualBox Version: 7.0
    ==> kubemaster: Setting hostname...
    ==> kubemaster: Configuring and enabling network interfaces...
    ==> kubemaster: Mounting shared folders...
        kubemaster: /vagrant => C:/Users/MSI BRAVO/kubernetes-install-cluster-with-kubeadm
    ==> kubenode01: Importing base box 'ubuntu/bionic64'...
    ==> kubenode01: Matching MAC address for NAT networking...
    ==> kubenode01: Setting the name of the VM: kubenode01
    ==> kubenode01: Fixed port collision for 22 => 2222. Now on port 2200.
    ==> kubenode01: Clearing any previously set network interfaces...
    ==> kubenode01: Preparing network interfaces based on configuration...
        kubenode01: Adapter 1: nat
        kubenode01: Adapter 2: hostonly
    ==> kubenode01: Forwarding ports...
        kubenode01: 22 (guest) => 2200 (host) (adapter 1)
    ==> kubenode01: Running 'pre-boot' VM customizations...
    ==> kubenode01: Booting VM...
    ==> kubenode01: Waiting for machine to boot. This may take a few minutes...
        kubenode01: SSH address: 127.0.0.1:2200
        kubenode01: SSH username: vagrant
        kubenode01: SSH auth method: private key
        kubenode01: Warning: Connection reset. Retrying...
        kubenode01: Warning: Connection aborted. Retrying...
        kubenode01: 
        kubenode01: Vagrant insecure key detected. Vagrant will automatically replace
        kubenode01: this with a newly generated keypair for better security.
        kubenode01: 
        kubenode01: Inserting generated public key within guest...
        kubenode01: Removing insecure key from the guest if it's present...
        kubenode01: Key inserted! Disconnecting and reconnecting using new SSH key...
    ==> kubenode01: Machine booted and ready!
    ==> kubenode01: Checking for guest additions in VM...
        kubenode01: The guest additions on this VM do not match the installed version of
        kubenode01: VirtualBox! In most cases this is fine, but in rare cases it can
        kubenode01: prevent things such as shared folders from working properly. If you see
        kubenode01: shared folder errors, please make sure the guest additions within the
        kubenode01: virtual machine match the version of VirtualBox you have installed on
        kubenode01: your host and reload your VM.
        kubenode01:
        kubenode01: Guest Additions Version: 5.2.42
        kubenode01: VirtualBox Version: 7.0
    ==> kubenode01: Setting hostname...
    ==> kubenode01: Configuring and enabling network interfaces...
    ==> kubenode01: Mounting shared folders...
        kubenode01: /vagrant => C:/Users/MSI BRAVO/kubernetes-install-cluster-with-kubeadm
    ==> kubenode02: Importing base box 'ubuntu/bionic64'...
    ==> kubenode02: Matching MAC address for NAT networking...
    ==> kubenode02: Setting the name of the VM: kubenode02
    ==> kubenode02: Fixed port collision for 22 => 2222. Now on port 2201.
    ==> kubenode02: Clearing any previously set network interfaces...
    ==> kubenode02: Preparing network interfaces based on configuration...
        kubenode02: Adapter 1: nat
        kubenode02: Adapter 2: hostonly
    ==> kubenode02: Forwarding ports...
        kubenode02: 22 (guest) => 2201 (host) (adapter 1)
    ==> kubenode02: Running 'pre-boot' VM customizations...
    ==> kubenode02: Booting VM...
    ==> kubenode02: Waiting for machine to boot. This may take a few minutes...
        kubenode02: SSH address: 127.0.0.1:2201
        kubenode02: SSH username: vagrant
        kubenode02: SSH auth method: private key
        kubenode02: Warning: Connection reset. Retrying...
        kubenode02: Warning: Connection aborted. Retrying...
        kubenode02: 
        kubenode02: Vagrant insecure key detected. Vagrant will automatically replace
        kubenode02: this with a newly generated keypair for better security.
        kubenode02: 
        kubenode02: Inserting generated public key within guest...
        kubenode02: Removing insecure key from the guest if it's present...
        kubenode02: Key inserted! Disconnecting and reconnecting using new SSH key...
    ==> kubenode02: Machine booted and ready!
    ==> kubenode02: Checking for guest additions in VM...
        kubenode02: The guest additions on this VM do not match the installed version of
        kubenode02: VirtualBox! In most cases this is fine, but in rare cases it can
        kubenode02: prevent things such as shared folders from working properly. If you see
        kubenode02: shared folder errors, please make sure the guest additions within the
        kubenode02: virtual machine match the version of VirtualBox you have installed on
        kubenode02: your host and reload your VM.
        kubenode02:
        kubenode02: Guest Additions Version: 5.2.42
        kubenode02: VirtualBox Version: 7.0
    ==> kubenode02: Setting hostname...
    ==> kubenode02: Configuring and enabling network interfaces...
    ==> kubenode02: Mounting shared folders...
        kubenode02: /vagrant => C:/Users/MSI BRAVO/kubernetes-install-cluster-with-kubeadm

Bạn có thể kiểm tra trạng thái các máy ảo đã dựng bằng lệnh sau:

    vagrant status

Thông tin về các máy ảo được tạo và quản lý bởi `vagrant` được trả về

    Current machine states:

    kubemaster                running (virtualbox)
    kubenode01                running (virtualbox)
    kubenode02                running (virtualbox)

    This environment represents multiple VMs. The VMs are all listed
    above with their current state. For more information about a specific
    VM, run `vagrant status NAME`.

#### Lỗi nhức nách: vagrant up times out tại bước 'default: SSH auth method: private key' 

Lỗi này xảy ra khi bật máy ảo không thành công. Mặc định, Virtual Box sử dụng TSC mode gọi là "RealTscOffset," để điều chỉnh giá trị TSC ([Time Stamp Counter](https://learning.oreilly.com/library/view/mastering-linux-kernel/9781785883057/20712c1b-f659-40da-a09d-55efc93b0597.xhtml)) trên máy ảo để đồng bộ clock freqency của CPU giữa máy host và máy ảo.

Nếu bạn đang sử dụng Windows đã bật phần mềm máy ảo `Hyper-V`, phải tắt `Hyper-V` để tránh gây ra xung đột với `Virtual Box` dẫn đến lỗi `vagrant up` time out ở trên.

Để tắt hoàn toàn `Hyper-V`, chạy lệnh sau trong cmd:

    bcdedit /set hypervisorlaunchtype off

sau đó tắt và bật lại máy tính. 

>Chú ý rằng `bcdedit` là viết tắt của `boot configuration data edit`, nói cách khác nó sẽ ảnh hưởng đến những phần mềm có thiết lập khi boot lại hệ điều hành, vì vậy bạn cần phải `shutdown` máy hoàn toàn (không `suspend` hay `restart`) để áp dụng thay đổi. Để PC tắt trong khoảng `10 giây` trước khi bật lại. Nếu PC không cho shutdown trong Start menu, bạn có thể chạy lệnh `shutdown /p` trong cmd dưới quyền admin. Trên laptop, bạn có thể cần phải tháo pin.

Cài lại `Vagrant` và `Virtual Box`. Nếu lỗi vẫn còn, có thể bạn sẽ phải cài lại hệ điều hành `Windows`, nhớ đừng bật `Hyper-V`!

## Truy cập vào máy ảo bằng Vagrant

Để ssh vào máy ảo, chỉ cần chạy lệnh: 

    vagrant ssh <hostname>

![Vagrant SSH in VSCode](../images/vagrant-ssh-vscode.png)

Có thể thấy trong output khi chạy `vagrant up`, `Vagrant` có chuyển tiếp port 22 và tạo ssh keypairs cho mỗi máy ảo dù chúng ta không thiết lập trong `Vagrantfile`. Để xem thêm thông tin, bạn có thể đọc [Vagrant Share: SSH Sharing](https://developer.hashicorp.com/vagrant/docs/share/ssh) và [Vagrantfile: config.ssh](https://developer.hashicorp.com/vagrant/docs/vagrantfile/ssh_settings).

OK, sang bước kế tiếp nào!

## Tiếp theo

▶️ [Cài đặt container runtime (containerd) trên tất cả các máy ảo](Installing-a-container-runtime.md)