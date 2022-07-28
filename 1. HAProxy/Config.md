# **File cấu hình HAProxy**

Cấu hình của HAProxy thường được tạo từ 4 thành phần bao gồm global, defaults, frontend, backend. 4 thành phần sẽ định nghĩa cách HAProxy nhận, xử lý các request, điều phối các request tới các Backend phía sau

- Đường dẫn file cấu hình: /etc/haproxy/haproxy.cfg

- Cấu trúc tổng quan:

|<p>**global**</br># Các thiết lập tổng quan</p><p>**defaults**<br># Các thiết lập mặc định</p><p>**frontend**<br># Thiết lập điều phối các request</p><p>**backend**<br> # Định nghĩa các server xử lý request</p>|
| :- |


- Khi khởi tạo dịch vụ haproxy, các thiết lập tại **global** sẽ được sử dụng để định nghĩa cách HAProxy được khởi tạo như số lượng kết nối tối đa, đường dẫn ghi file log, số process v.v.
 
- Sau đó các thiết lập tại mục **defaults** sẽ được áp dụng cho tất cả mục **frontend**, **backend** nằm phía sau (có thể định nghĩa lại các giá trị mặc định tại **frontend** và **backend**)
 
- Có thể có nhiều mục **frontend**, **backend** được định nghĩa trong file cấu hình.
 
- Mục **frontend** được định nghĩa để điều hướng các request nhận được tới các **backend**. Mục **backend** sử dụng để định nghĩa các danh sách máy chủ dịch vụ (có Web server, Database, …) đây là nơi request được xử lý

***

**Global**

- Mục này luôn đứng riêng 1 dòng và được định nghĩa 1 lần duy nhất trong file cấu hình
 
- Các thiết lập bên dưới global định nghĩa các thiết lập bảo mật, các điều chỉnh về hiệu năng áp dụng trên toàn HAProxy (áp dụng tại mức tiến trình HAProxy hoạt động)
 
- Ví dụ:

```
global
    log         127.0.0.1 local2
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon
    stats socket /var/lib/haproxy/stats
```


- **log**: các cảnh báo phát sinh tại HAProxy trong quá trình khởi động, vận hành sẽ được gửi tới syslog
 
- **maxconn**: Chỉ định giới hạn số kết nối mà HAProxy có thể thiết lập. Sử dụng với mục đích bảo vệ load balancer khởi vấn đề tràn ram
 
- **user / group:** chỉ định quyền sử dụng để khởi tạo tiến trình HAProxy. Linux yêu cầu xử lý bằng quyền root cho những port nhỏ hơn 1024. Nếu không định nghĩa user và group, HAProxy sẽ tự động sử dụng quyền root khi thực thi tiến trình.
 
- **stats socket:** Cho phép kết nối tới socket HAProxy API. Định nghĩa runtime api, có thể sử dụng để disable server hoặc health checks, thay đổi load balancing weights của server

***

**Defaults**

- Khi cấu hình tăng dần, phức tạp, khó đọc, các thiết lập cấu hình tại mục defaults giúp giảm các trùng lặp. Thiết lập tại mục defaults sẽ áp dụng cho tất cả mục frontend backend nằm sau nó. 
 
- Có thể thiết lập lại trong từng mục backend, frontend. Và cũng có thể có nhiều mục defaults. Chúng sẽ ghi đè lên nhau dựa theo vị trí (tức các mục defaults nằm sau sẽ ghi đè lên các mục defaults nằm trước
 
- Ví dụ đơn giản, thế thiết lập mode http tại mục defaults, khi đó toàn bộ các mục frontend, backend, listen sẽ đều dùng mode http làm mặc định.
 

```
mode                    http
log                     global
option                  httplog
option                  dontlognull
option                  http-server-close
option forwardfor       except 127.0.0.0/8
option                  redispatch
retries                 3
option  http-server-close
timeout http-request    10s
timeout queue           1m
timeout connect         10s
timeout client          1m
timeout server          1m
timeout http-keep-alive 10s
timeout check           10s
maxconn                 3000
```

- **timeout connect** chỉ định thời gian HAProxy đợi thiết lập kết nối TCP tới backend server. Hậu tố s tại 10s thể hiện khoảng thời gian 10 giây, nếu bạn không có hậu tố s, khoảng thời gian sẽ tính bằng milisecond
 
- **timeout server** chỉ định thời gian chờ kết nối tới backend server.
 
- Khi thiết lập **mode tcp** thời gian **timeout server** phải bằng **timeout client**
 
- **log global:** Chỉ định ‘**frontend**’ sẽ sử dụng log settings mặc định (trong mục global)
 
- **mode**: Thiết lập mode định nghĩa HAProxy sẽ sử dụng TCP proxy hay HTTP proxy. Cấu hình sẽ áp dụng với toàn **frontend** và **backend** khi chỉ mong muốn sử dụng 1 mode mặc định trên toàn **backend** (Có thể thiết lập lại giá trị tại backend)
 
- **maxconn**: Thiết lập chỉ định số kết nối tối đa, mặc định bằng 2000.
 
- **option httplog:** Bổ sung format log dành riêng cho các request http bao gồm (connection timers, session status, connections numbers, header v.v). Nếu sử dụng cấu hình mặc định các tham số sẽ chỉ bao gồm địa chỉ nguồn và địa chỉ đích
 
- **option http-server-close**: Khi sử dụng kết nối dạng keep-alive, tùy chọn cho phép sử dụng lại các đường ống kết nối tới máy chủ (có thể kết nối đã đóng) nhưng đường ống kết nối vẫn còn tồn tại, thiết lập sẽ giảm độ trễ khi mở lại kết nối từ phía client tới server.
 
- **option dontlognull:** Bỏ qua các log format không chứa dữ liệu
 
- **option forwardfor:** Sử dụng khi mong muốn backend server nhận được IP thực của người dùng kết nối tới. Mặc định backend server sẽ chỉ nhận được IP của HAProxy khi nhận được request. Header của request sẽ bổ sung thêm trường X-Forwarded-For khi sử dụng tùy chọn
 
- **option redispatch:** Trong mode HTTP, khi sử dụng kỹ thuật stick session, client sẽ luôn kết nối tới 1 backend server duy nhất, tuy nhiên khi backend server xảy ra sự cố, có thể client không thể kết nối tới backend server khác (Trong bài toán load balancer). Sử dụng kỹ thuật cho phép HAProxy phá vỡ kết nối giữa client với backend server đã xảy ra sự cố. Đồng thời, client có thể khôi phục lại kết nối tới backend server ban đầu khi dịch vụ tại backend server đó trở lại hoạt động bình thường.
 
- **retries:** Số lần thử kết nối lại backend server trước khi HAProxy đánh giá backend server xảy ra sự cố.
- **timeout check:** Kiểm tra thời gian đóng kết nối (chỉ khi kết nối đã được thiết lập)
- **timeout http-request:** Thời gian chờ trước khi đóng kết nối HTTP
- **timeout queue:** Khi số lượng kết nối giữa client và haproxy đạt tối đã (maxconn), các kết nối tiếp sẽ đưa vào hàng đợi. Tùy chọn sẽ làm sạch hàng chờ kết nối.

***

**Frontend**

- Mục **frontend** định nghĩa địa chỉ IP và port mà client có thể kết nối tới. Có thể có nhiều mục frontend tùy ý, chỉ cần đặt label của chúng khác nhau (frontend <tên>)
 
- Ví dụ:

```
frontend www.mysite.com
    bind 10.0.0.3:80
    bind 10.0.0.3:443 ssl crt /etc/ssl/certs/mysite.pem
    http-request redirect scheme https unless { ssl_fc }
    use_backend api_servers if { path_beg /api/ }
    default_backend web_servers
```

- **bind:** IP và Port HAProxy sẽ lắng nghe để mở kết nối. IP có thể bind tất cả địa chỉ sẵn có hoặc chỉ 1 địa chỉ duy nhất, port có thể là một port hoặc nhiều port (1 khoảng hoặc 1 list).
- **http-request redirect:** Phản hỏi tới client với đường dẫn khác. Ứng dụng khi client sử dụng http và phản hồi từ HAProxy là https, điều hướng người dùng sang giao thức https
- **use\_backend**: Chỉ định backend sẽ xử lý request nếu thỏa mãn điều kiện (Khi sử dụng ACL)
- **default\_backend:** Backend mặc định sẽ xử lý request (Nếu request không thỏa mẵn bất kỳ điều hướng nào)

***

**Backend**

- Mục backend định nghĩa tập server sẽ được cân bằng tải khi có các kết nối tới (VD tập các server chạy dịch vụ web giống nhau).
 
- Ví dụ:

```
backend web_servers
    balance roundrobin
    cookie SERVERUSED insert indirect nocache
    option httpchk HEAD /
    default-server check maxconn 20
    server server1 10.10.10.86:80 cookie server1
    server server2 10.10.10.87:80 cookie server2
```

- **balance**: Kiểm soát cách HAProxy nhận, điều phối request tới các backend server. Đây chính là các thuật toán cân bằng tải.
- **cookie:** Sử dụng cookie-based. Cấu hình sẽ khiến HAProxy gửi cookie tên SERVERUSED tới client, liên kết backend server với client. Từ đó các request xuất phát từ client sẽ tiến tục nói chuyện với server chỉ định. Cần bổ sung thêm tùy chọn cookie trên server line
- **option httpchk:** Với tùy chọn, HAProxy sẽ sử dụng health check dạng HTTP (Layer 7) thay vì kiếm trả kết nối dạng TCP (Layer 4). Và khi server không phản hồi request http, HAProxy sẽ thực hiện TCP check tới IP Port. Health check sẽ tự động loại bỏ các backend server lỗi, khi không có backend server sẵn sàng xử lý request, HAProxy sẽ trả lại phản hồi 500 Server Error. Mặc đinh HTTP check sẽ kiểm tra root path / (Có thể thay đổi). Và nếu phản hồi health check là 2xx, 3xx sẽ được coi là thành công.
- **default-server:** Bổ sung tùy chọn cho bất kỳ backend server thuộc backend section (VD: health checks, max connections, v.v). Theo ví dụ, tùy chọn maxconn 20 sẽ được bổ sung vào tất cả backend server, tức mỗi server sẽ chỉ phục vụ 20 kết nối đồng thời
- **server:** Tùy chọn quan trọng nhất trong backend section. Tùy chọn đi kèm bao gồm tên, IP:Port. Có thể dùng domain thay cho IP.

***

**Listen**

- **listen** là sự kết hợp của cả 2 mục frontend và backend. Vì listen kết hợp cả 2 tính năng backend frontend, nên có thể sử dụng listen thay thế các các mục backend và frontend.
 
- Ví dụ:

```
listen web-backend
    bind 10.10.10.89:80
    balance leastconn
    cookie SERVERID insert indirect nocache
    mode  http
    option  forwardfor
    option  httpchk GET / HTTP/1.0
    option  httpclose
    option  httplog
    timeout  client 3h
    timeout  server 3h
    server node1 10.10.10.86:80 weight 1 check cookie s1
    server node2 10.10.10.87:80 weight 1 check cookie s2
    server node3 10.10.10.88:80 weight 1 check cookie s3
```


- Cú pháp thường dùng:

```
listen web-backend:
    ...
    server node1 10.10.10.86:80 inter <time> rise <number> fall <number>
```


- **inter:** khoảng thời gian giữa hai lần check liên tiếp.
- **rise:** Số lần kiểm tra backend server thành công trước khi HAProxy đánh giá nó đang hoạt động bình thường và bắt đầu điều hướng request tới
- **fall:** Số lần kiểm tra backend server bị tính là thất bại trước khi HAProxy đánh giá nó xảy ra sự cố và không điều hướng request tới.



- **Một cấu hình cơ bản:**

```
global
  log stdout format raw local0 info
  user haproxy 
  group haproxy
  stats socket /var/run/api.sock mode 660 level admin expose-fd listeners
     # Phần cấu hình HAProxy API: 
     # stats socket /var/run/haproxy.sock mode 600 expose-fd listeners level user
    # stats socket ipv4@127.0.0.1:9999 level admin
    # stats timeout 2m
    # server-state-file /var/lib/haproxy/server-state

defaults
  mode http
  timeout client 10s
  timeout connect 5s
  timeout server 10s
  timeout http-request 10s
  log global
  # Phần cấu hình HAProxy API:
     # load-server-state-from-file global

frontend stats
  bind *:8404
  stats enable
  stats uri /
  stats refresh 10s

frontend myfrontend
  bind :80
  default_backend webservers

backend webservers
  server s1 web1:8000 check
  server s2 web2:8001 check
```

- Trong phần Global, stats socket kích hoạt HAProxy Runtime API và seamless reloads của HAProxy
 
- /var/run/api.sock: Đường dẫn tới file socket kết nối tới
 
- ipv4@127.0.0.1:9999: Chỉ định địa chỉ và port được phép kết nối tới HAProxy API
 
- **server-state-file** và **load-server-state-from-file**: Lưu lại trạng thái server. 
  Ví dụ, khi chuyển trạng thái server thành maintain nhưng đây chỉ là trạng thái tạm thời. Đến khi khởi động lại HAProxy, trạng thái server đó sẽ trở lại trạng thái active. Để tránh tình trạng đó xảy ra, sử dụng 2 cấu hình **server-state-file** và **load-server-state-from-file**
 
- frontend đầu tiên sử dụng cổng 8404 để vào bảng điều khiển HAProxy Stats, hiển thị thống kê trực tiếp về bộ cân bằng tải.
 
- frontend thứ hai sử dụng cổng 80 để gửi yêu cầu đến một trong hai webservers được liệt kê trong phần backend webservers





- **Một trong những cấu hình mặc định thường dùng cho HAProxy:**

```
global
    log         127.0.0.1 local2
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon
    stats socket /var/lib/haproxy/stats

defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

listen stats
    bind :8080
    mode http
    stats enable
    stats uri /stats
    stats realm HAProxy\ Statistics

listen webcluster
    bind :80
    balance  roundrobin
    mode  http
    option  forwardfor
    server web1 10.10.11.87:80 check
    server web2 10.10.11.88:80 check' > /etc/haproxy/haproxy.cfg
```


- Khi chỉnh sửa file cấu hình haproxy.cfg, có thể reload lại load balancer mà không làm gián đoạn lưu lượng truy cập:

`docker kill -s HUP haproxy`
