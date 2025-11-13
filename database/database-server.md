# Tài nguyên database-server

| Hostanme |  IP      | OS       | CPU (tối thiểu) | RAM (tối thiểu) |  Role        |  
|----------|----------------|------------|--------------|----------|------------------|  
| database-server | 192.168.1.115 | Ubuntu 22.04   | 1            | 2      |database   |  
 
# Cài đặt
## Cài đặt mariadb

```bash
$ nano apt install mariadb-server
```

## mở kết nối cho bên ngoài kết nối vào mariadb

```bash
$ nano /etc/mysql/mariadb.conf.d/50-server.cnf

```

```sh
bind-address = 0.0.0.0
```

## Copy file dự án bên ngoài vào database-server
```bash
$ scp .\Fullstack-Ecommerce-Web.zip ubuntu@192.168.1.115:/tmp

```

