# Tài nguyên rancher-server

| Hostanme |  IP      | OS       | CPU (tối thiểu) | RAM (tối thiểu) |  Role        |  
|----------|----------------|------------|--------------|----------|------------------|  
| rancher-server | 192.168.1.114 | Ubuntu 22.04   | 1            | 2      | dashboard     |  
 
# Cài đặt

## thêm hosts
```bash  
$ nano /etc/hosts
```

```sh
192.168.1.113 rancher-server

```

## Gán disk (ổ cứng) vào trong thư mục /data
```bash  
$ sudo mkfs.ext4 -m 0 /dev/sdb
$ mkdir /data
$ echo "/dev/sdb  /data  ext4  defaults  0  0" | sudo tee -a /etc/fstab
$ mount -a
$ sudo df -h
```

## File docker-compose.yml tạo Rancher

```bash  
$ nano Dockerfile

```

```sh
version: '3'
services:
  rancher-server:
    image: rancher/rancher:v2.9.2
    container_name: rancher-server
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /data/rancher/data:/var/lib/rancher
    privileged: true
```

## Lấy mật khẩu Rancher
```bash
$ docker logs rancher-server 2>&1 | grep "Bootstrap Password:"
```

