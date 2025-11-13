# Tài nguyên của cụm Kubernetes  

| Hostanme |  IP      | OS       | CPU (tối thiểu) | RAM (tối thiểu) |  Role        |  
|----------|----------------|------------|--------------|----------|------------------|  
| k8s-master-1 | 192.168.1.111 | Ubuntu 22.04   | 2            | 3      | Master/Node     |  
| k8s-master-2 | 192.168.1.112 | Ubuntu 22.04   | 2            | 3      | Master/Node     |  
| k8s-master-3 | 192.168.1.113 | Ubuntu 22.04   | 2            | 3      | Master/Node     |  

# Cài đặt
## Thực hiện trên tất cả servers

### thêm hosts
```bash  
$ nano /etc/hosts
```

```sh
192.168.1.111 k8s-master-1
192.168.1.112 k8s-master-2
192.168.1.113 k8s-master-3
```
### Tạo user devops và chuyển sang user devops
```bash  
$ adduser devops
$ su devops
$ /home/devops 
```
### Tắt swap
```bash  

sudo swapoff -a
sudo sed -i '/swap.img/s/^/#/' /etc/fstab

```

### Cấu hình module kernel
```bash
$ nano /etc/modules-load.d/containerd.conf

```

```sh
overlay
br_netfilter
```

### Tải module kernel
```bash
$ sudo modprobe overlay
$ sudo modprobe br_netfilter
```

### Cấu hình hệ thống mạng

```bash
$ echo "net.bridge.bridge-nf-call-ip6tables = 1" | sudo tee -a /etc/sysctl.d/kubernetes.conf
$ echo "net.bridge.bridge-nf-call-iptables = 1" | sudo tee -a /etc/sysctl.d/kubernetes.conf
$ echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.d/kubernetes.conf
```

### Áp dụng cấu hình sysctl
```bash
$ sudo sysctl --system

```

### Cài đặt các gói cần thiết và thêm kho Docker
```bash
$ sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

### Cài đặt containerd
```bash
$ sudo apt update -y
$ sudo apt install -y containerd.io

```

### Cấu hình containerd

```bash
$ containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
$ sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

```
### Khởi động containerd
```bash
$ sudo systemctl restart containerd
$ sudo systemctl enable containerd

```

### Thêm kho lưu trữ Kubernetes
```bash
$ echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

```

### Cài đặt các gói Kubernetes
```bash
$ sudo apt update -y
$ sudo apt install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl
```

## Mô hình cụm 1 master 2 worker
### Thực hiện trên server k8s-master-1
```bash
$ sudo kubeadm init
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
$ kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml

```
### Thực hiện trên k8s-master-2, k8s-master-3
```bash
$ sudo kubeadm join 192.168.1.111:6443 --token your_token --discovery-token-ca-cert-hash your_sha
```