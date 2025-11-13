# 🚀 Kubernetes On-Premise HA Cluster (6 Nodes) Deployment

Dự án triển khai **Kubernetes High Availability on-premise** với kiến trúc 3 master, load balancer HA, Rancher Server quản lý cluster và database server riêng. Dự án này phục vụ mục đích học tập & mô phỏng môi trường doanh nghiệp thực tế.

---

## 🏗️ Kiến trúc hệ thống

Cluster gồm 6 node vật lý:

| Node | Vai trò | IP Address |
|------|----------|------------|
| Master 1 | Kubernetes Control Plane | 192.168.1.111 |
| Master 2 | Kubernetes Control Plane | 192.168.1.112 |
| Master 3 | Kubernetes Control Plane | 192.168.1.113 |
| Load Balancer | HAProxy / Nginx LB cho API Server | 192.168.1.110 |
| Rancher Server | UI quản lý Kubernetes | 192.168.1.114 |
| Database Server | MariaDB / PostgreSQL (cho Rancher / App) | 192.168.1.115 |

### 🔗 Kết nối Control Plane thông qua Load Balancer
API Server endpoint:
```
https://192.168.1.110:6443
```

### ☸️ Thành phần chính
- Kubernetes: v1.xx.x (triển khai bằng kubeadm)
- Container Runtime: containerd
- CNI: Calico (BGP mode)
- Load Balancer: HAProxy / Nginx
- Management UI: Rancher
- Database: MariaDB / PostgreSQL
- Monitoring stack: Prometheus + Grafana (optional)
- Logging: Loki (optional)

---

## 🖼️ Sơ đồ kiến trúc (Architecture Diagram)

*(Bạn thêm file PNG hoặc tui thiết kế cho bạn nếu muốn)*

---

## 📂 Cấu trúc thư mục repository

```
k8s-onprem-HA/
│── README.md
│── docs/
│     ├── architecture.png
│     ├── setup-notes.md
│── manifests/
│     ├── namespaces/
│     ├── deployments/
│     ├── services/
│     ├── ingress/
│── rancher/
│     ├── docker-install.sh
│     ├── rancher-values.yaml
│── scripts/
│     ├── init-master.sh
│     ├── join-master.sh
│     ├── join-worker.sh
│     ├── install-containerd.sh
│── loadbalancer/
│     ├── haproxy.cfg
│     ├── nginx-lb.conf
│── database/
│     ├── create-db.sql
│     ├── backup-script.sh
```

---

## ⚙️ Triển khai Kubernetes Control Plane

### 1️⃣ Cài containerd
```
./scripts/install-containerd.sh
```

### 2️⃣ Init control plane qua Load Balancer
```
kubeadm init \
 --control-plane-endpoint "192.168.1.110:6443" \
 --upload-certs \
 --pod-network-cidr=192.168.0.0/16
```

### 3️⃣ Join master 2 & master 3
```
kubeadm join 192.168.1.110:6443 --control-plane --token <token> ...
```

---

## 🌐 Cài đặt CNI (Calico)
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---

## 🛠️ Cài đặt Load Balancer (HAProxy)
File: `loadbalancer/haproxy.cfg`

```
frontend kubernetes
    bind *:6443
    default_backend k8s-masters

backend k8s-masters
    balance roundrobin
    server master1 192.168.1.111:6443 check
    server master2 192.168.1.112:6443 check
    server master3 192.168.1.113:6443 check
```

---

## 🐮 Rancher Server (Node: 192.168.1.114)
```
docker run -d --restart=unless-stopped \
 -p 80:80 -p 443:443 \
 rancher/rancher:latest
```

---

## 🗄️ Database Server (192.168.1.115)
Ví dụ MariaDB:
```
sudo systemctl enable mariadb
sudo systemctl start mariadb
mysql_secure_installation
```

---

## 📦 Triển khai ứng dụng demo
```
kubectl apply -f manifests/deployments/demo-app.yaml
kubectl apply -f manifests/services/demo-service.yaml
```

---

## 📊 Kết quả đạt được
- Triển khai thành công **Kubernetes HA cluster 3 master + load balancer**  
- Rancher quản lý toàn bộ cluster  
- Tự động hóa cài đặt thông qua script  
- Tách biệt database riêng để mô phỏng môi trường enterprise  

---

## 👤 Tác giả
**Tên bạn**  
📌 GitHub: https://github.com/your-username  
📌 Email: your-email@example.com

