# **Practise**

**Task:** Xây dựng hệ thống dịch vụ có cân bằng tải (load balancing) và tính khả dụng cao (highly available) sử dụng HAProxy + Keepalived. Tất cả được Deploy bằng Docker

**Keywords:** HAProxy, Keepalived, Docker, Docker-compose, nginx, server, Load Balance,  highly available

***

**Mô tả hệ thống:**

- Có 2 máy chủ web là: **web-a** và **web-b**. Sử dụng Nginx làm máy chủ web (đáp ứng cả HTTP và HTTPS)
- Trước máy chủ web có 2 HAProxy: **haproxy-a** và **haproxy-a**, 
  - 2 HAProxy kết nối với hai máy chủ web để đảm bảo hiệu năng. 
 
  - Bên cạnh đó keepalived được cài đặt để kiểm tra xem HAProxy có khả dụng không và đặt 1 IP ảo để client sử dụng. Định nghĩa **host 1** là node chính (master) còn **host 2** là node phụ (backup) để hỗ trợ chuyển đổi dự phòng HAProxy 

![viettel drawio](https://user-images.githubusercontent.com/43572616/182453155-a00ce4af-c027-452a-a08b-549dcabd76a1.png)

Link: https://drive.google.com/file/d/1W2x1ObqEYL3jx6fQDpgGRYzyyH2EDCPp/view

- **Virtual Server:** 
  - **IP:** 192.168.12.145
  - **Port H2:** 8404
  - **Port HTTP:** 80





- **vtnet network:** 
  - **IP:** 172.20.0.0/24
  - haproxy-a: 172.20.0.50
  - haproxy-b: 172.20.0.60
  - web-a, web-b



- **Host 1:**
  - **Hostname:** ansible\_target\_1
  - **IP:** 192.168.23.143/24
  - **Interface:** ens33
  - **Instances:**
    - keepalived-a
    - haproxy-a
    - haproxy-b
    - web-a
    - web-b



- **Host 2:**
  - **Hostname:** ansible\_target\_2
  - **IP:** 192.168.23.144/24
  - **Interface:** ens33
  - **Instances:**
    -  keepalived-b
    - haproxy-a
    - haproxy-b
    - web-a
    - web-b



- **Máy client:**
  - **Hostname:** ansible\_controller
  - **IP:** 192.168.23.142/24
  - **Interface:** ens33

***

**Set-up**

- **Bật ip_vs**

```
yum install ipvsadm
modprobe ip_vs
ipvsadm
```
***
- **Configure system:** [Link](<https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/load_balancer_administration/s1-initial-setup-forwarding-vsa>)

```
sysctl -w net.ipv4.ip\_forward=1
sysctl -w net.ipv4.ip\_nonlocal\_bind=1
```

***

- **Thêm port:**

```sh
firewall-cmd --add-port=80/tcp
firewall-cmd --add-port=8404/tcp
```

***

**Run docker-compose:**

- **Host 1:** 

`docker compose up -d keepalived-a haproxy-a haproxy-b web-a web-b`

- **Host 2:** 

`docker compose up -d keepalived-b haproxy-a haproxy-b web-a web-b`

***

**Test hệ thống**

Ở backend dùng retries 2 nên HAProxy sẽ sử dụng thay đổi web-a và web-b liên tục 

- **Trên host 1 và host 2:**
  - Truy cập trực tiếp vào 2 HAProxy:

![image](https://user-images.githubusercontent.com/43572616/182454075-9ac0410f-bb2b-4d2e-bdd9-e436dd772cbe.png)
![image](https://user-images.githubusercontent.com/43572616/182454096-5854434a-9fa5-4805-8d95-9f226e19188f.png)


- Thông qua VIP, Keepalived:

![image](https://user-images.githubusercontent.com/43572616/182454122-0fcca4b4-2b9b-4d16-91d6-05847a1c0e1a.png)

- Health Check:

![image](https://user-images.githubusercontent.com/43572616/182454156-d3b548b6-706b-48c5-84c3-33a65c453ee8.png)

- Giả sử web-a sập:

![image](https://user-images.githubusercontent.com/43572616/182454177-9a6b26d5-32b4-4b35-94ce-852fbce4984a.png)

Gọi HAProxy:

![image](https://user-images.githubusercontent.com/43572616/182454203-e0f5e349-08cd-4dc9-9b11-0624ee731585.png)

Gọi thông qua VIP, Keepalived:

![image](https://user-images.githubusercontent.com/43572616/182454215-8b4f42d9-4ab2-4dd6-830c-b9d7bd818319.png)

Khôi phục lại web-a

![image](https://user-images.githubusercontent.com/43572616/182454244-042a994d-7e8e-4455-a077-9124cb11a44e.png)

Chạy bình thường

![image](https://user-images.githubusercontent.com/43572616/182454266-2cf12b9e-36ab-4ce9-ade2-917ea1a5a320.png)
![image](https://user-images.githubusercontent.com/43572616/182454290-978f9258-5426-4995-9a8d-045c7fe6d189.png)

- Giả sử haproxy-a sập:

![image](https://user-images.githubusercontent.com/43572616/182454307-245b329a-4ad8-4f66-a71b-2ea024e76e62.png)

Gọi haproxy-a:

![image](https://user-images.githubusercontent.com/43572616/182454334-7fae200a-44f5-45d3-85b2-6bc0b95ca654.png)

Gọi haproxy-b, chạy bình thường:

![image](https://user-images.githubusercontent.com/43572616/182454356-77f2f33a-6fde-407a-a90e-802823f547df.png)

Gọi thông qua VIP, Keepalived - chạy bình thường:

![image](https://user-images.githubusercontent.com/43572616/182454372-1e0fa4af-9418-49ab-8f4d-ba5214a6144e.png)

Khôi phục lại haproxy-a:

![image](https://user-images.githubusercontent.com/43572616/182454401-f6047558-0bfc-48ee-919b-85e40754c87e.png)

Chạy bình thường:

![image](https://user-images.githubusercontent.com/43572616/182454423-471ccbf2-608e-48be-b4bf-297cafc56417.png)

***

- **Trên máy khách (Client):**
  - Truy cập web:

![image](https://user-images.githubusercontent.com/43572616/182454456-c6c43eeb-812d-43a8-94a3-a5068c9ba2d5.png)

- Health Check:

![Untitled](https://user-images.githubusercontent.com/43572616/182454492-a52d3121-e59f-4dd2-b573-4ad4bf28e6f9.png)

- Giả sử host 2 sập:

![image](https://user-images.githubusercontent.com/43572616/182454513-aa616264-8e8a-4e1b-bcd2-ad810dc8155b.png)

Truy cập bình thường:

![image](https://user-images.githubusercontent.com/43572616/182454539-91c25801-7c12-41ee-85ed-0112a2fd2401.png)

Thử Health Check bình thường:

![image](https://user-images.githubusercontent.com/43572616/182454555-a9b2287e-3a86-4b5b-ba6e-a42f01d34b16.png)

- Giả sử cả 2 host đều sập:

![image](https://user-images.githubusercontent.com/43572616/182454570-147c946e-0520-434e-833b-e81ff0eb8551.png)

Không thể truy cập:

![image](https://user-images.githubusercontent.com/43572616/182454587-3e603bdb-d218-4d1b-83c8-747a4b198cdb.png)
