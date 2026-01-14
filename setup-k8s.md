# Setup Cluster K8s
## Setup chung cho Master và Worker Node
### Prepare Linux
#### Update linux repository
```bash
apt update -y
```

#### Disable Swap
```bash
swapoff -a
vi /etc/fstab
```
Remove swap from /etc/fstab file

### Install Container Runtime
#### Tải các module kernel cần thiết:
```bash
printf "overlay\nbr_netfilter\n" >> /etc/modules-load.d/containerd.conf
modprobe overlay
modprobe br_netfilter
```
Lệnh đầu tiên thêm các module overlay và br_netfilter vào file cấu hình /etc/modules-load.d/containerd.conf để đảm bảo chúng sẽ được tải khi khởi động hệ thống.

modprobe overlay và modprobe br_netfilter được sử dụng để tải ngay lập tức các module này vào kernel. Module overlay hỗ trợ hệ thống file OverlayFS, cần thiết cho containerd, và module br_netfilter cho phép kiểm soát lưu lượng mạng giữa các container.

#### Cấu hình các thông số mạng hệ thống:
```bash
printf "net.bridge.bridge-nf-call-iptables = 1\nnet.ipv4.ip_forward = 1\nnet.bridge.bridge-nf-call-ip6tables = 1\n" > /etc/sysctl.d/99-kubernetes-cri.conf
sysctl --system
```
Thêm các cấu hình liên quan đến mạng vào file /etc/sysctl.d/99-kubernetes-cri.conf:
* net.bridge.bridge-nf-call-iptables = 1 : Cho phép iptables kiểm soát lưu lượng mạng qua các bridge.
* net.ipv4.ip_forward = 1 : Cho phép chuyển tiếp IP, cần thiết để forward các gói IP giữa các mạng.
* net.bridge.bridge-nf-call-ip6tables = 1 : Tương tự như iptables, nhưng cho IPv6.

#### Cài đặt containerd:
```bash
wget https://github.com/containerd/containerd/releases/download/v1.7.13/containerd-1.7.13-linux-amd64.tar.gz -P /tmp
tar Cxzvf /usr/local /tmp/containerd-1.7.13-linux-amd64.tar.gz
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -P /etc/systemd/system/
systemctl daemon-reload
systemctl enable --now containerd
```

#### Cài đặt runc:
```bash
wget https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64 -P /tmp/
install -m 755 /tmp/runc.amd64 /usr/local/sbin/runc
```

#### Cài đặt các plugin mạng CNI:
```bash
wget https://github.com/containernetworking/plugins/releases/download/v1.4.0/cni-plugins-linux-amd64-v1.4.0.tgz -P /tmp
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin /tmp/cni-plugins-linux-amd64-v1.4.0.tgz
```
Tạo thư mục /opt/cni/bin và giải nén các plugin vào đó.
Các plugin này cung cấp các chức năng mạng cơ bản cho Kubernetes, như cấu hình địa chỉ IP, routing, và các quy tắc iptables.

#### Cấu hình containerd:
```bash
mkdir -p /etc/containerd
containerd config default | tee /etc/containerd/config.toml
vi /etc/containerd/config.toml
```
Tạo thư mục cấu hình /etc/containerd.

containerd config default | tee /etc/containerd/config.toml tạo file cấu hình mặc định cho containerd.

Sử dụng vi để chỉnh sửa file /etc/containerd/config.toml, thay đổi giá trị SystemdCgroup thành true ở dòng 137 để containerd sử dụng cgroup của systemd, giúp quản lý tài nguyên hệ thống một cách hiệu quả hơn.

### Install Dependency
Dependency need to install: docker, kubeadm, kubelet
* Add GPG key for Kubernetes repository
```bash
apt-get install ca-certificates curl gnupg lsb-release apt-transport-https gpg
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

* Add Kubernetes repository
```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

* Update repository
```bash
apt update -y
reboot
```

* Install dependency components
```bash
apt install -y kubelet=1.29.1-1.1 kubeadm=1.29.1-1.1 kubectl=1.29.1-1.1
```

* Hold Kubernetes components version
```bash
apt-mark hold kubelet kubeadm kubectl
```

* Enable kubelet service
```bash
systemctl enable --now kubelet
```
