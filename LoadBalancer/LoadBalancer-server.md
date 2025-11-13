# Tài nguyên Load Balancer-server

| Hostanme |  IP      | OS       | CPU (tối thiểu) | RAM (tối thiểu) |  Role        |  
|----------|----------------|------------|--------------|----------|------------------|  
| LoadBalancer-server | 192.168.1.110 | Ubuntu 22.04   | 1            | 2      | dashboard     |  

# Cài đặt

## Cài đặt Helm
```bash
$ wget https://get.helm.sh/helm-v3.16.2-linux-amd64.tar.gz
$ tar xvf helm-v3.16.2-linux-amd64.tar.gz
$ sudo mv linux-amd64/helm /usr/bin/
$ helm version

```

## Cài đặt Ingress controller`
```bash
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm repo update
$ helm pull ingress-nginx/ingress-nginx
$ tar -xzf ingress-nginx-4.11.3.tgz
$ vi ingress-nginx/values.yaml
>> Sửa type: LoadBalancing => type: NodePort
>> Sửa nodePort http: "" => http: "30080"
>> Sửa nodePort https: "" => https: "30443"
$ kubectl create ns ingress-nginx
$ helm -n ingress-nginx install ingress-nginx -f ingress-nginx/values.yaml ingress-nginx

```
## Thay đổi port server
```bash
$ nano /etc/nginx/sites-available/default

```

## Cấu hình nginx server Load balancer
```bash
$ nano /etc/nginx/conf.d/devopsedu.vn.conf

```

```sh
upstream my_servers {
    server 192.168.1.111:30080;
    server 192.168.1.112:30080;
    server 192.168.1.113:30080;
}

server {
    listen 80;

    location / {
        proxy_pass http://my_servers;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```