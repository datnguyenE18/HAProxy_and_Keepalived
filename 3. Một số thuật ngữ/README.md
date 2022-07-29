# Một số thuật ngữ

- **Proxy Server:**
  - Proxy Server (Máy chủ proxy) hoạt động như một cổng nối giữa người dùng và Internet. Đây là một server trung gian giữa người dùng cuối và trang web họ truy cập
</br>
  - Nếu đang sử dụng máy chủ proxy, lưu lượng truy cập Internet sẽ truyền qua máy chủ proxy theo đường của nó đến địa chỉ yêu cầu. Sau đó, yêu cầu này sẽ trở lại cùng một máy chủ proxy (có thể có ngoại lệ) và máy chủ proxy đó sẽ chuyển tiếp dữ liệu nhận được từ website đến người dùng.

![server proxy](https://user-images.githubusercontent.com/43572616/177666783-e76c133b-2c17-4c92-8fab-1385e21f61bd.jpg)

- **Proxy Server vận hành:**
  - Một proxy server về cơ bản là một máy tính trên mạng internet với địa chỉ IP riêng mà máy tính của người dùng có thể nhận biết được
</br>
  - Khi gửi đi một web request, request đó sẽ tới proxy server trước tiên. Proxy server sau đó sẽ thay bạn thực hiện yêu cầu web, nhận các phản hồi từ web server và chuyển người dùng đến trang web dữ liệu để có thể xem trang trong trình duyệt.
</br>
  - Proxy server có thể chặn truy cập một trang web nhất định:

![203925226](https://user-images.githubusercontent.com/43572616/177666873-73b00f7f-b7be-4e69-8d0a-0a57846461e6.jpg)

***

- **Access Control List (ACL):**
  - Trong cân bằng tải, Acess Control List (ACL) được sử dụng để kiểm tra điều kiện và thực hiện một hành động (VD: chọn một server hay chặn một request) dựa trên kết quả của việc kiểm tra đó.

  - Khi sử dụng ACL cho phép tạo một môi trường có khả năng chuyển tiếp các request linh hoạt dựa trên các yếu tố khác nhau.

  - Ví dụ một ACL:
    `acl url_blog src /something`

Trong đó: ACL này sử dụng cho các request có chứa /something.

***

- **Backend:**
  - Backend là một tập các server mà HAProxy có thể chuyển tiếp các request tới. Backend được cấu hình trong mục backend trong file configuration của HAProxy. Backend có thể cài đặt bằng cách:

    - Đặt thuật toán cân bằng tải (round-robin, least-connection,…)
    - Danh sách các máy chủ và port có thể nhận request từ HAProxy.
</br>
  - Một backend có thể chứa một hoặc nhiều server, về cơ bản thì càng nhiều server thì khả năng chịu tải và performace của hệ thống càng tăng. HAProxy cho phép một server backup chuyên dụng, được sử dụng khi các server offline.
</br>
  - Ví dụ về cấu hình backend:

    ```sh
    backend web-backend
      balance leastconn
      mode http
      server backend-1 web-backend-1.example.com check
      server backend-2 web-backend-2.example.com check
      server backend-3 backup-backend.example.com check backup

    backend forum
      balance leastconn
      server forum-1 forum-1.example.com check
      server forum-2 forum-2.example.com check
      server forum-3 backup-forum.example.com check backup
    ```

Trong đó:

- Dòng blance leaseconn chỉ ra thuật toán cân bằng tải là chọn các server có ít kết nối đến nó nhất.
</br>
- Dòng mode http chỉ ra rằng các proxy sẽ chỉ cân bằng cho các kết nối tại tầng 7 của Internet Layer.

***

- **Frontend**
  - Fontend được sử dụng để định nghĩa cách mà các request điều hướng cho backend. Và được định nghĩa trong mục fontend của HAProxy configuration. Các cấu hình cho frontend gồm:
    - Một địa chỉ IP và port.
    - Các ACL do người dùng định nghĩa.
    - Backend được sử dụng để nhận các request.
  - Ví dụ về cấu hình fontend:

    ```sh
    frontend web
      bind 0.0.0.0
      default_backend web-backend

    frontend forum
      bind 0.0.0.0:8080
      default_backend forum
    ```

***

- **Cluster:**  là việc kết hợp nhiều phần cứng riêng lẻ thành một cụm để cung cấp dịch vụ. Cluster có thể là Active-Active hay Active-Standby tùy theo từng hãng

  - Ví dụ Microsoft SQL Server là A-S, Oracle RAC là A-A. Cluster có thể là các máy để cạnh nhau trong data center hoặc các máy nằm xa nhau (HP có công nghệ continenal cluster)
</br>
  - Ưu điểm của A-S là chi phí thấp và dễ triển khai, dễ quản trị.
</br>
  - Ưu điểm của A-A là vừa giải quyết được vấn đề tính sẵn sàng và vừa chia được tải, nhược điểm là khó quản trị, chi phí phần cứng đắt, năng lực tổng không tăng tuyến tính theo số node mà sẽ tăng chậm dần. Thường Cluster A-A trên 4 node chỉ tăng được 200%-300% so với 1 node tùy theo công nghệ mỗi hãng

***

- **Failover :** là khả năng khi một node bị sự cố, giao dịch sẽ được tiếp tục phục vụ trên 1 node khác trong cluster mà client không phải thực hiện lại từ đầu
</br>
- Để có thể failover được thì NLB (Network Load Balancer) chỉ giải quyết được phần ngọn, còn phần gốc là toàn bộ các thông tin giao dịch trong bộ nhớ của node sự cố phải được replicate sang các node khác

***

**HTTP vs HTTPS:**

- HTTPS là viết tắt của từ HyperText Transfer Protocol Secure và chính là giao thức HTTP có sử dụng thêm các chứng chỉ SSL (secure Sockets Layer) giúp mã hóa dữ liệu truyền tải nhằm gia bảo mật giữa Web sever đến các trình duyệt web. Nói cách khác HTTPS là phiên bản HTTP nhưng an toàn hơn, bảo mật hơn.

***
**Runtime API:** Có trong (embedded) bộ cân bằng tải (load balancer) mà không cần cài thêm packages nào khác. Nó cho phép sửa một số cấu hình của load balancer mà không cần khởi động lại dịch vụ. Những thay đổi được lưu trong memory cho đến khi tải/khởi động lại dịch vụ mà không làm thay đổi file cấu hình trên đĩa. Sử dụng tính năng này để thực hiện những thay đổi nhanh như bật/tắt server hoặc thêm đường dẫn (entries) để map file vào memory

  **Org:** The Runtime API is embedded inside the load balancer and requires no other packages. It lets you configure some aspects of the load balancer at runtime without needing to reload the service. Changes are kept in memory only until the next reload or restart and are not saved to the configuration file on disk. Use this feature to make on-the-fly changes, such as enabling and disabling servers or adding entries to map files in memory.

***
**seamless reload:** Thường được gọi là “seamless” hay “hitless” reload, là một cấu hình (configuration) mà khi cấu hình hoặc nâng cấp dịch vụ không làm ảnh hưởng đến trải nghiệm người dùng

***
**Máy chủ upstream ( upstream server ):** là một máy chủ cung cấp các dịch vụ cho các máy chủ khác và nằm cao hơn/bên trên so với hệ thống các máy chủ.

***
**SSL/TLS:** TLS trước đây là SSL là giao thức mật mã được thiết kế để cung cấp truyền thông an toàn qua một mạng máy tính

***
**SSL Certificate (Chứng chỉ SSL):**

- Là một tập tin nhỏ được mã hóa chứa dữ liệu thông tin của một website hoặc một tổ chức/công ty.
</br>
- Khi cài đặt chứng chỉ lên máy chủ website (webserver), nó sẽ cho phép website sử dụng kết nối an toàn (hay còn gọi là Giao thức HTTPS) khi giao tiếp giữa webserver và trình duyệt của người dùng. Khi dữ liệu truyền đi thông qua giao thức HTTPS, các dữ liệu sẽ được mã hóa và chỉ có webserver chứa khóa riêng (private key) thì mới có thể giải mã được dữ liệu này.
</br>
  - Để có chứng chỉ SSL, khách hàng sẽ cần phải thực hiện chứng thực để chứng minh là người sở hữu tên miền hợp lệ. Với các chức chỉ dành cho doanh nghiệp thì việc chứng thực sẽ phức tạp hơn vì tổ chức chứng thực sẽ cần biết chắc chắn doanh nghiệp của bạn đang tồn tại và hoạt động. Chính vì lẽ đó mà các website có chứng chỉ SSL sẽ có độ uy tín tốt hơn tùy thuộc vào loại chứng chỉ đang sử dụng
</br>
  - Khi sử dụng giao thức HTTPS, trình duyệt sẽ hiển thị địa chỉ website với dạng https:// và có hình ổ khóa
</br>
  - **Site Seal:** Khi sử dụng chứng chỉ SSL trả phí, thì mỗi chứng chỉ sẽ được cấp một logo chứng nhận là đang sử dụng chứng chỉ SSL của tổ chức đó, hay còn được gọi là Site Seal hoặc Trust Seal. Gồm có 2 loại:
</br>
    - **Dynamic:** Là site seal dạng động, có thể hỗ trợ hiệu ứng hiển thị thông tin về website hoặc doanh nghiệp khi rê chuột vào. Điều này sẽ giúp người dùng tin tưởng vào website mà họ đang truy cập nhiều hơn
</br>
    - **Static:** Với một số chứng chỉ SSL giá rẻ thì chỉ hỗ trợ site seal dạng tĩnh, nó chỉ là một tấm ảnh và có thể đặt vào bất cứ đâu trong website nhưng sẽ không có hiệu ứng rê chuột vào hoặc không thể nhấp để xem thông tin được

***

- **Multicast:**
  - là thuật ngữ được sử dụng để mô tả cách thức truyền tin gửi từ 1 điểm đến 1 tập hợp các điểm khác, là một hình thức kết nối đa điểm. Kỹ thuật này được sử dụng theo mô hình OSI trong tầng mạng
</br>
  - Lợi thế của nó là cùng một lúc thông điệp được gởi tới nhiều người tham gia hoặc tới một nhóm người sử dụng kín. Máy gửi chỉ cần tốc độ truyền tải dữ liệu giống như khi chỉ có một máy nhận duy nhất trong khi multicasting. Multicast hữu ích nếu 1 nhóm khách hàng yêu cầu 1 bộ dữ liệu chung cùng 1 lúc. Việc truyền tin Multicast sẽ tiết kiệm băng thông một cách đáng kể

***

- **Real Server:** Virtual server là một server ảo được xây dựng trên nhiều server thực, các server thực này dùng chung một public ip, do đó tương tác giữa người dùng với tập hợp server này không khác gì với một server bình thường. Request của người dùng được gởi đến load balancer (public ip của load balancer này là ip chung cho cả hệ thống server), load balancer sẽ chọn ra một real server và chuyển gói tin đến real server đó để xử lí

***

- **LVS - Linux Virtual Server**: là kỹ thuật nhóm các server đơn lẻ thành một server duy nhất mà người dùng không hề biết về sự tồn tại của các server đơn lẻ này. Server duy nhất này được gọi là Virtual Server hay còn gọi là bộ cân bằng tải – load balancer có một địa chỉ IP ảo (Virtual IP hay VIP) và đằng sau nó là các Real Server
