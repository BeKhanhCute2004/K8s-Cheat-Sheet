## Prepare Linux
### Update linux repository
```bash
apt update -y
```

### Disable Swap
```bash
swapoff -a
vi /etc/fstab
```
Remove swap from /etc/fstab file

## Install Container Runtime
### Tải các module kernel cần thiết:
```bash
printf "overlay\nbr_netfilter\n" >> /etc/modules-load.d/containerd.conf
modprobe overlay
modprobe br_netfilter
```
Lệnh đầu tiên thêm các module overlay và br_netfilter vào file cấu hình /etc/modules-load.d/containerd.conf để đảm bảo chúng sẽ được tải khi khởi động hệ thống.

modprobe overlay và modprobe br_netfilter được sử dụng để tải ngay lập tức các module này vào kernel. Module overlay hỗ trợ hệ thống file OverlayFS, cần thiết cho containerd, và module br_netfilter cho phép kiểm soát lưu lượng mạng giữa các container.
