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
#### Add GPG key for Kubernetes repository
```bash
apt-get install ca-certificates curl gnupg lsb-release apt-transport-https gpg
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

#### Add Kubernetes repository
```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

#### Update repository
```bash
apt update -y
reboot
```

#### Install dependency components
```bash
apt install -y kubelet=1.29.1-1.1 kubeadm=1.29.1-1.1 kubectl=1.29.1-1.1
```

#### Hold Kubernetes components version
```bash
apt-mark hold kubelet kubeadm kubectl
```
apt-mark hold (Giữ phiên bản): Đây là lệnh bạn thấy trong hình. Khi bạn "hold" một gói tin, hệ thống sẽ khóa phiên bản hiện tại của gói đó lại. Khi bạn chạy apt upgrade, các gói này sẽ không bao giờ bị tự động cập nhật lên bản mới hơn.
Tại sao Kubernetes cần cái này? Trong cụm K8s, các thành phần như kubelet, kubeadm cần sự đồng nhất về phiên bản giữa các node. Nếu một node tự ý cập nhật lên bản mới hơn các node còn lại, cụm máy chủ có thể bị lỗi hoặc mất ổn định.

#### Enable kubelet service
```bash
systemctl enable --now kubelet
```

### Bootstraping
#### Bootstraping master using kubeadm
* Bootstrap cơ bản cho test
```bash
kubeadm init --pod-network-cidr 192.168.0.0/16
```
Lưu ý: khi init lần đầu nên dùng với cờ --dry-run để test vì init là lỗi một chiều không thể quay lại được phải xoá hết làm lại từ đầu. Ở đây nếu không rành nên để đường mạng mặc định như trên để cấu hình hợp với Calico bên dưới.

* Bootstrap đầy đủ 
```bash
sudo kubeadm init \
  --control-plane-endpoint "<IP_OR_DOMAIN:PORT>" \
  --pod-network-cidr "<POD_CIDR>" \
  --service-cidr "<SERVICE_CIDR>" \
  --upload-certs \
  --apiserver-cert-extra-sans "<IP_OR_DOMAIN>,<IP_OR_DOMAIN>,..."
```

Ý nghĩa các tham số trên:
- --pod-network-cidr: Định nghĩa dải địa chỉ IP cho các Pod. Ví dụ: nếu bạn dùng Calico, thường sẽ đặt là 192.168.0.0/16.
- --service-cidr: Định nghĩa dải địa chỉ IP cho các Service (các IP ảo để truy cập ứng dụng).
- --upload-certs: Tự động tải các chứng chỉ bảo mật lên cụm để nếu bạn có thêm máy Master thứ 2 hoặc thứ 3, chúng có thể tự đồng bộ chứng chỉ về. Secret chứa các chứng chỉ này chỉ tồn tại trong 2 giờ. Nếu quá thời gian này bạn mới join thêm máy Master, bạn sẽ phải chạy lệnh để upload lại. (`kubeadm init phase upload-certs --upload-certs`)
- --apiserver-cert-extra-sans: Thêm các tên miền hoặc IP bổ sung vào chứng chỉ SSL của API Server (giúp bạn có thể điều khiển cluster từ xa một cách an toàn qua các IP khác). Lời khuyên: nên add vào IP + hostname của tất cả Master Node, IP Load Balance, Domain name public, IP mặt ngoài (public).
- --control-plane-endpoint : xác định ip của load balancer làm điểm kết nối duy nhất cho toàn bộ cluster. tất cả các máy worker sẽ nhìn vào địa chỉ này để liên lạc với hệ thống điều khiển thay vì gọi trực tiếp vào từng ip của master. Nếu muốn chỉ load balance mặt ngoài thì không cần cờ này, chỉ cần đổi IP trong file config và thêm --apiserver-cert-extra-sans.

Các tham số cấu hình thêm:
- --node-name <STRING>: đặt tên cho host (mặc định lấy host name nếu không set)
- --apiserver-advertise-address <ENDPOINT>: xác định ip thật của máy master hiện tại để api server mở cổng (bind) và lắng nghe traffic đổ vào. nó giúp định danh chính xác dịch vụ đang chạy trên card mạng nào của máy đó.


#### Configure Token
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Mục đích chính của các câu lệnh này là cấp quyền cho người dùng bình thường (không phải root) có thể sử dụng lệnh kubectl để quản lý cụm Cluster.
Sau khi thiết lập xong, Master Node sẽ trong trạng thái not ready do chưa có Pod Network nào

```bash
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
https://kubernetes.io/docs/concepts/cluster-administration/addons/
```

#### Lựa chọn 1: Cài đặt Flannel (Đơn giản nhất)
```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

#### Lựa chọn 2: Cài đặt Calico (Phổ biến, bảo mật cao)
```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/tigera-operator.yaml
sleep 10
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/custom-resources.yaml
```
create khác apply ở chỗ từ lần chạy thứ 2 apply sẽ tự động ghi đè còn create kiểm tra thấy thì sẽ báo lỗi => nên dùng apply hơn

#### Worker Node join Cluster
* Join Cluster
```bash
kubeadm join 172.26.9.10:6443 --token 551p5f.95n94jwhkh7zxi0y \
    --discovery-token-ca-cert-hash sha256:c7da60f995cfbbd72f98b80378dd8bc048bdc8849148a6d1d90636dda716fd31
```
* nếu quên lưu hoặc token hết hạn có thể dùng lệnh này
```bash
kubeadm token create --print-join-command
```

#### Master Node join Cluster
* Join Cluster
```bash
kubeadm join kubernetes.kbuor.tech:6443 --token mn5dts.0tjj19w4z8bduz8k \
  --discovery-token-ca-cert-hash sha256:daec4eb2cb544a59ed57c4e9a7675b56827b69210024fb4d5d6e850a5bc88218 \
  --control-plane --certificate-key ae7198e6201490fdbd32f68db1d045dc80ee5d684b0ee04b03ccfb8d27a7ef98
```
* nếu quên lưu hoặc token hết hạn có thể dùng lệnh này
```bash
sudo kubeadm init phase upload-certs --upload-certs

kubeadm token create --print-join-command --certificate-key <mã-key-vừa-lấy-ở-bước-1>
```

## Cluster Upgrades

### Upgrade Commands For First Master Node
```bash
# Check current version
kubectl version

# Add new repository
apt-get install ca-certificates curl gnupg lsb-release apt-transport-https gpg
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/`enter new version`/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Update repository
apt update -y

# Upgrade kubeadm
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm && \
apt-mark hold kubeadm

# Plan upgrade
kubeadm upgrade plan

# Apply upgrade
kubeadm upgrade apply `new version`

# Upgrade node
kubeadm upgrade node

# Upgrade kubelet and kubectl
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=`new version` kubectl=`new version` && \
apt-mark hold kubelet kubectl

# Restart kubelet
systemctl daemon-reload
systemctl restart kubelet
```

### Upgrade Commands For Other Master Node
```bash
# Check current version
kubectl version

# Add new repository
apt-get install ca-certificates curl gnupg lsb-release apt-transport-https gpg
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/`enter new version`/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Update repository
apt update -y

# Upgrade kubeadm
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm && \
apt-mark hold kubeadm

# Upgrade node
kubeadm upgrade node

# Upgrade kubelet and kubectl
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=`new version` kubectl=`new version` && \
apt-mark hold kubelet kubectl

# Restart kubelet
systemctl daemon-reload
systemctl restart kubelet
```

### Upgrade Commands For Other Master Node
```bash
# Check current version
kubectl version

# Drain node (It includes cordon)
kubectl drain <tên-worker-node> --ignore-daemonsets

# Add new repository
apt-get install ca-certificates curl gnupg lsb-release apt-transport-https gpg
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/`enter new version`/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Update repository
apt update -y

# Upgrade kubeadm
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm && \
apt-mark hold kubeadm

# Upgrade node
kubeadm upgrade node

# Upgrade kubelet and kubectl
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=`new version` kubectl=`new version` && \
apt-mark hold kubelet kubectl

# Restart kubelet
systemctl daemon-reload
systemctl restart kubelet

# Uncordon node
kubectl uncordon <tên-worker-node>
```
---

## Installation Metal LB + Nginx
### Install Metal LB
#### Install template Metal LB by Manifest
This will deploy MetalLB to your cluster, under the metallb-system namespace. The components in the manifest are:
* The metallb-system/controller deployment. This is the cluster-wide controller that handles IP address assignments.
* The metallb-system/speaker daemonset. This is the component that speaks the protocol(s) of your choice to make the services reachable.
* Service accounts for the controller and speaker, along with the RBAC permissions that the components need to function.

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.15.3/config/manifests/metallb-native.yaml
```

#### Configure Metal LB (metallb-config.yaml)
```bash
kubectl create namespace metallb-system
```
```bash
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: metallb-pool
  namespace: metallb-system
spec:
  addresses:
  - 10.20.30.200-10.20.30.210  # dải IP được chọn
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2-advertisement
  namespace: metallb-system
spec: {}
```
```bash
kubectl apply -f metallb-config.yaml
```

#### Setup NGINX Ingress by Helm 
Điều kiện, phải cài đặt Helm trên Node Controller, chạy lệnh dưới trên Controller luôn.
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install [RELEASE_NAME] ingress-nginx/ingress-nginx
```
(Cài đặt Helm nếu chưa có)
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh
```
#### Check Ingress Controller Running
```bash
kubectl get pods -n ingress-nginx
```
#### Create Deployment For Test (app.yaml)
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
      - name: demo
        image: hashicorp/http-echo:0.2.3
        args:
        - "-text=Hello from Demo App"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: demo-service
spec:
  selector:
    app: demo
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678
  type: ClusterIP
```
```bash
kubectl apply -f app.yaml

