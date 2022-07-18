# **Practise**


**Task:** Xây dựng hệ thống dịch vụ có cân bằng tải (load balancing) và tính khả dụng cao (highly available) sử dụng HAProxy + Keepalived. Tất cả được Deploy bằng Docker

**Keywords:** HAProxy, Keepalived, Docker, Docker-compose, nginx, server, Load Balance,  highly available

***

**Hệ thống dự kiến:**

- Có 2 máy chủ web là: **web1** và **web2**. Sử dụng Nginx làm máy chủ web (đáp ứng cả HTTP và HTTPS)
- Trước máy chủ web có 2 load balancers: **bl1** và **bl2**, 
  - 2 HAProxy kết nối với hai máy chủ web để đảm bảo hiệu năng. 
 
  - Bên cạnh đó keepalived được cài đặt để kiểm tra xem HAProxy có khả dụng không và đặt 1 IP ảo để khách sử dụng. Định nghĩa **bl1** là node chính (master) còn **bl2** là node phụ (backup) để hỗ trợ chuyển đổi dự phòng HAProxy 

![177694292-4e5d2130-1722-4aa2-859f-58d371a5c476](https://user-images.githubusercontent.com/43572616/178408860-a150cc19-d5af-465f-bdb7-a54fd70e79f9.png)


***


**Mô tả hệ thống hoàn thiện**

![viettel drawio](https://user-images.githubusercontent.com/43572616/178408901-7d5e5ac0-8daf-486e-b643-0097e3e660f0.png)


- **Virtual Server:** 192.168.0.150
- **vtnet network:** 172.20.0.0
  - **haproxy-a:** 172.20.0.50
  - **haproxy-b:** 172.20.0.60
  - **Health Check port:** 8404
  - web-a & web-b
- **Interface:** ens33

***

**Test hệ thống**

Ở backend dùng retries 2 nên HAProxy sẽ sử dụng thay đổi web-a và web-b liên tục 

- **Truy cập trực tiếp vào 2 HAProxy:**

![image](https://user-images.githubusercontent.com/43572616/178408957-3bba26bb-7373-4596-8e92-3f110d67e036.png)



![image](https://user-images.githubusercontent.com/43572616/178408966-198560a2-b40b-4bca-90bb-6bf99b59739c.png)



- **Thông qua VIP, Keepalived:**

![image](https://user-images.githubusercontent.com/43572616/178408977-b5e4187f-7480-4962-9e92-9deeac23447b.png)



- **Health Check:**

![image](https://user-images.githubusercontent.com/43572616/178409008-a9044a6a-e247-4ab3-ab5f-aa5d4bbcf148.png)



- **Giả sử web-a sập:**

![image](https://user-images.githubusercontent.com/43572616/178409023-7b54ca95-1762-4b93-a290-76a8d527d40a.png)



Gọi HAProxy:

![image](https://user-images.githubusercontent.com/43572616/178409037-a4b36bd2-7157-4b6e-b5ff-08f9dd4d6304.png)



Gọi thông qua VIP, Keepalived:

![image](https://user-images.githubusercontent.com/43572616/178409044-51b2f9cc-6a8a-42d8-bd05-0d55971d7e96.png)



Khôi phục lại web-a

![image](https://user-images.githubusercontent.com/43572616/178409055-4caa29dd-25f5-4c8a-8b65-3d2873716e6a.png)



Chạy bình thường

![image](https://user-images.githubusercontent.com/43572616/178409062-dcb1591c-7da5-46b9-8f58-7ecbcc492421.png)



![image](https://user-images.githubusercontent.com/43572616/178409076-ede322c4-97d1-4832-8b4d-0d80bb00f4ca.png)



- **Giả sử haproxy-a sập:**

![image](https://user-images.githubusercontent.com/43572616/178409086-582ed560-b5a5-4d8c-8197-04ea5320ac40.png)



Gọi haproxy-a:

![image](https://user-images.githubusercontent.com/43572616/178409104-d0d519a1-7ce9-459a-aaab-669c89c8a22a.png)



Gọi haproxy-b, chạy bình thường:

![image](https://user-images.githubusercontent.com/43572616/178409126-6ba6a398-9992-4531-b03e-564c078102ae.png)



Gọi thông qua VIP, Keepalived - chạy bình thường:

![image](https://user-images.githubusercontent.com/43572616/178409138-629667ef-5b27-4936-b150-31582538f546.png)



Khôi phục lại haproxy-a:

![image](https://user-images.githubusercontent.com/43572616/178409149-23e96339-9146-4cbe-bb88-1ce662db37d6.png)



Chạy bình thường:

![image](https://user-images.githubusercontent.com/43572616/178409159-b4c5d5b9-f375-44ef-b3e2-db1d8187fc19.png)


