# 🚀 Kubernetes On-Premise HA Cluster (6 Nodes) Deployment

Dự án triển khai **Kubernetes High Availability on-premise** với kiến trúc 1 master và 2 worker, load balancer Nginx, Rancher Server quản lý cluster và database server riêng. Dự án này phục vụ mục đích học tập & mô phỏng môi trường doanh nghiệp thực tế.

---

## 🏗️ Kiến trúc hệ thống

Cluster gồm 6 node vật lý:

| Node | Vai trò | IP Address |
|------|----------|------------|
| Master 1 | Kubernetes Control Plane | 192.168.1.111 |
| Master 2 | Kubernetes Control Plane | 192.168.1.112 |
| Master 3 | Kubernetes Control Plane | 192.168.1.113 |
| Load Balancer | Nginx LB cho API Server | 192.168.1.110 |
| Rancher Server | UI quản lý Kubernetes | 192.168.1.114 |
| Database Server | MariaDB (cho Rancher) | 192.168.1.115 |

### 🔗 Kết nối Control Plane thông qua Load Balancer
API Server endpoint:
```
https://192.168.1.110:30080
```

### ☸️ Thành phần chính
- Kubernetes: v1.30.14 (triển khai bằng kubeadm)
- Container Runtime: containerd
- CNI: Calico (BGP mode)
- Load Balancer: Nginx
- Management UI: Rancher
- Database: MariaDB 

---


## 🖼️ Sơ đồ kiến trúc (Architecture Diagram)
<img width="1024" height="622" alt="Screenshot 2025-11-13 142117" src="https://github.com/user-attachments/assets/51db012b-cd01-4f75-b369-365c7165ad62" />



## 📂 Cấu trúc thư mục repository

```
k8s-onprem-HA/
│── README.md
│── infrastructure
│     ├── manifests/
|         ├── Install K8S
|             ├── k8s.md  
│         ├── namespaces/
|             ├── ecommerce
│         ├── deployments/
|             ├── ecommerce-frontend-deplpoyment
|             ├── ecommerce-backend-deployment    
│         ├── services/
|             ├── ecommerce-frontend-service 
|             ├── ecommerce-backend-serivce  
│         ├── ingress/
|             ├── ecommerce-frontend-ingress 
|             ├── ecommerce-backend-ingress
│──  rancher/
│      ├── rancher-server.md
|── LoadBalancer
|      ├── LoadBalancer-seriver.md
├── database/
|      ├──  Fullstack-Ecommerce-Web/
|             ├──1-starter-files_db-scripts
|             ├── 02-backend_spring-boot-rest-api
|             ├── 03-frontend_angular-ecommerce
├── Fullstack-Ecommerce-Web/
|      ├── 01-starter-files_db-scripts
|      ├── 02-backend_spring-boot-rest-api
|      ├── 03-frontend_angular-ecommerce

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
 --control-plane-endpoint "192.168.1.111:30080" \
 --upload-certs \
 --pod-network-cidr=192.168.0.0/16
```

### 3️⃣ Join master 2 & master 3
```
kubeadm join 192.168.1.112:30080 --control-plane --token <token> ...
kubeadm join 192.168.1.113:30080 --control-plane --token <token> ...
```

---

## 🌐 Cài đặt CNI (Calico)
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---

## 🛠️ Cài đặt Load Balancer (Nginx)
File: `loadbalancer/nginx.cfg`

```
frontend kubernetes
    bind *:80
    default_backend k8s-masters

backend k8s-masters
    balance roundrobin
    server master1 192.168.1.111:30080 check
    server master2 192.168.1.112:30080 check
    server master3 192.168.1.113:30080 check
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
sudo systemctl enable mariadb
sudo systemctl start mariadb
mysql_secure_installation
## 📦 Triển khai ứng dụng ecommerce
```
Build Images
docker build -t ecommerce-frontend:v1 .
docker build -t ecommerce-backend:v1 .
Tag images lên dockerhub
docker tag ecommerce-frontend:v1 locdevops/ecommerce-frontend:v1
docker tag ecommerce-backend:v1 locdevops/ecommerce-backend:v1
```
```
Apply namespace ->  deployment -> service -> ingress
kubectl create ns ecommerce
kubectl apply -f manifests/deployments/ecommerce-frontend-deployment.yaml
kubectl apply -f manifests/services/ecommercce-frontend-service.yaml
kubectl apply -f manifests/ingress/ecommerce-frontend-ingress.yaml
kubectl apply -f manifests/deployments/ecommerce-backend-deployment.yaml
kubectl apply -f manifests/services/ecommerce-backend-service.yaml
kubectl apply -f manifests/ingress/ecommerce-backend-ingress.yaml

```
<img width="1915" height="868" alt="Screenshot 2025-11-13 185559" src="https://github.com/user-attachments/assets/c1b476e0-3de3-4afe-aa6c-ae98169b2e14" />


---

## 📊 Kết quả đạt được
- Triển khai thành công **Kubernetes HA cluster 1 master + 2 worker + load balancer**  
- Rancher quản lý toàn bộ cluster  
- Tự động hóa cài đặt thông qua script  
- Tách biệt database riêng để mô phỏng môi trường enterprise

<img width="1919" height="868" alt="Screenshot 2025-11-13 185740" src="https://github.com/user-attachments/assets/3ed5e456-690e-460e-9000-13fe63bd4aba" />

---

## 👤 Tác giả
Huỳnh Tấn Lộc
📌 GitHub: https://github.com/huynhtanloc0908  
📌 Email: tanlochuynh112@gmail.com

