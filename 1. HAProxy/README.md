# HAProxy

- **HAProxy** là một proxy server, load balancer, system monitoring, được sinh ra để làm nhiệm vụ của một front end web service.

  Là một phần mềm sử dụng trong việc cân bằng tải, chạy trên Linux, Solaris và FreeBSD. Người dùng có thể sử dụng HAProxy để cải thiện suất hoàn thiện của các trang web và ứng dụng bằng cách phân tán khối lượng công việc của chúng trên nhiều máy chủ.
</br>
- HAProxy cũng được sử dụng trong các hệ thống lớn có lưu lượng truy cập cao như GitHub, Twitter, Reddit, Bitbucket, Stack Overflow,…

![177666641-5dfc3b55-718a-4002-9b14-465ef62204c8](https://user-images.githubusercontent.com/43572616/178094092-b4dbe20d-8a5a-4634-854e-cdc6bb742171.png)

***

- **Khi nào sử dụng HAProxy**

  - **Ví dụ:** Khi  làm việc với dự án nhỏ, web app chỉ cần viết bằng NodeJS dùng Pm2 để có thể chạy multithread sau đó dùng Nginx để serve static file, làm proxy cho nodejs là được. Rất ít khi lỗi, quá tải vì cơ bản là một app nhỏ, nếu sập cũng chỉ cần restart lại
</br>
      Nhưng khi bước vào một công ty, phải đối mặt với những vấn đề mới, khi lượng request lên tới hàng ngìn request/s thì việc scale hệ thống như thế nào để có thể đáp ứng được lượng request đó là một vấn đề cần phải giải quyết.
</br>
      Về cơ bản, khi lượng request nhiều lên, sẽ có thể lựa chọn hai loại hình scale cho hệ thống đó là scale ngang và scale dọc
</br>
  - Scale dọc tức là nâng cấp HDD lên SSD dung lượng cao, nâng cấp RAM, Vi xử lí… (đắt, chi phí cao)
</br>
  - Scale theo chiều ngang tức là dùng thêm nhiều hệ thống tương tự như hệ thống hiện tại để chia nhau xử lí request.

      Thường khi scale sẽ kết hợp cả hai kiểu scale sao cho hợp lí nhất và ít chi phí nhất. Đến đây khi các hệ thống hoạt động cùng với nhau, nảy sinh ra vấn đề làm sao để giao request này cho hệ thống kia xử lí

    => Vì thế Haproxy sinh ra để giải quyết vấn đề này, haproxy sẽ làm vai trò của một proxy server, theo dõi tình trạng các node và sẽ gửi request đến các node

***

- **Tính năng của HAProxy:**
  - Hỗ trợ cân bằng tải ở lớp 4 và lớp 7 (tương ứng với TCP và HTTP).
  - Support HTTP protocol, HTTP / 2, gRPC, FastCGI.
  - **SSL / TLS termination**
    - HAProxy sẽ mã hóa thông tin giao tiếp giữa client và chính nó, sau đó gửi thông tin đã được giải mã tới backend servers. Nghĩa là sẽ làm giảm tải việc mã hóa cho servers.
</br>
    - **Org:** A TLS termination proxy (or SSL termination proxy, or SSL offloading) is a proxy server that acts as an intermediary point between client and server applications, and is used to terminate and/or establish TLS (or DTLS) tunnels by decrypting and/or encrypting communications.

      The term SSL termination means that you are performing all encryption and decryption at the edge of your network, such as at the load balancer. The load balancer strips away the encryption and passes the messages in the clear to your servers. You might also hear this called SSL offloading

      When you route traffic through an HAProxy load balancer, you gain the ability to terminate SSL at the load balancer. HAProxy encrypts communication between the client and itself and then sends the decrypted messages to your backend servers, which means less CPU work on the servers because there’s no encryption work left to do
  - Lưu trữ chứng chỉ SSL động.
  - **Chuyển đổi nội dung và kiểm tra (Content switching and inspection)**
    - Là chuyển lưu lượng request của client từ một server ban đầu được lựa chọn bởi load balancing sang server khác dựa vào tiêu đề và nội dung của request

      Nghĩa là nó có quyền truy cập vào các requests/responses của ứng dụng. Rồi kiểm tra từng requests/responses sau đó nó đưa ra quyết định về nơi gửi những yêu cầu đó đến và cách xử lý phản hồi.
</br>
    - **Org:** Content switching is a programmatic redirection of client request traffic from one server, initially selected by load balancing, to another server that is selected based on the header or content of the request.

      Content-switching is an OSI layer 7 application switching technique, which in lamans terms means that it has access to the HTTP requests and responses of your application. By inspecting each request and response, it make decision on where to send those requests and how to handle responses.

      HAProxy is an open source load balancing tool that also has the ability to implement content switching.
</br>
  - **Đảm bảo trong suốt (Transparent proxying):**
    - Nếu nhà cung cấp bật logging trong HAProxy thì có thể dễ dàng xem logs truy cập của người dùng/khách hàng (bao gồm cả địa chỉ IP) thông qua câu lệnh: ```docker logs <tên HAProxy Container>```

    - Vì vậy HAProxy có thể có thể được cấu hình để ẩn địa chỉ IP của máy khách khi thiết lập kết nối TCP với máy chủ và máy chủ nghĩ rằng đang thực hiện kết nối trực tiếp đến từ máy khách. Bằng cách thêm dòng ```http-request set-var(txn.src_masked) src,ipmask(number)``` trong frontend tại HAProxy Config file
  </br>
  - Ghi nhật ký chi tiết.
  - Tương tác servers bằng dòng lệnh (CLI for server management)
  - Xác thực HTTP.
  - **Đa luồng:** Nhiều người dùng cùng lúc, xử lý nhiều yêu cầu cùng lúc
  - **Rewrite URL:** ghi lại địa chỉ website từ dạng này thành một dạng khác.
  - Kiểm tra sức khỏe nâng cao.
  - Giới hạn tần số kết nối.

***

- **Bảo mật:**
  - HAProxy được coi là an toàn, nó có rất ít lỗ hổng trong các năm qua
</br>
  - Nó chứa các tính năng có thể hạn chế tấn công như cô lập chính nó sử dụng chroot, drop ngay user/group không có các quyền đặc biệt khi khởi động và tránh truy cập vào ổ cứng khi khởi động.
</br>
  - HAProxy cũng có thể được sử dụng để bảo mật cho các hệ thống khác, ví dụ như: HAProxy theo dõi lưu lượng truy cập và giám sát hành vi của khách hàng thông qua các yêu cầu, sau đó có thể chặn khách hàng đó nếu thấy khả nghi. Người dùng có thể cấu hình ACL, xác định các chính sách để kiểm tra dữ liệu của các truy cập. Nó cũng có thể giới hạn tốc độ và danh sách IP Blacklist/Whitelist

***

- **Sticky session:**
  - Trong môi trường web, nhiều khi chúng ta cần cố định session của user, như để duy trì trạng thái login. Khi đó, chúng ta cần cố định session trên một server
</br>
  - HAProxy hỗ trợ một số thuật toán Load Balancing duy trì trạng thái kết nối mà cho phép cố định session như hdr, rdp-cookie, source, uri hoặc url_param. Ví dụ:

    ```sh
    backend cms
        balance source
        hash-type consistent
        server web1 192.168.10.110:8080 check
        server web2 192.168.10.111:8080 check
        server web3 192.168.10.112:8080 check
    ```

  - Nếu muốn cố định session mà vẫn sử dụng các thuật toán load balancing như roundrobin, leastconn, hoặc static-rr, khi đó sử dụng “Sticky Session”. Sticky session cho phép cố định session của users mà sử dụng cookie, và HAProxy sẽ điều phối để luôn request từ một user đến cùng một server.
</br>
  - HAProxy có thể lưu cookie trong trình duyệt của người dùng để ghi nhớ máy chủ nào sẽ gửi lại cho họ. Cookie sẽ chứa unique ID của server và được đảm bảo chỉ thuộc về một người dùng duy nhất, sẽ cung cấp thông tin chính xác để thực hiện cố định phiên (sticky sessions).
</br>
  - Để sử dụng sticky session trong HAProxy, chúng ta thêm tùy chọn “cookie cookie_name insert/prefix” vào trong phần backend. Chỉnh sửa tệp cấu hình HAProxy: /etc/haproxy/haproxy.cfg . Để tạo cookie trong trình duyệt của người dùng, hãy thêm "cookie" vào phần phụ trợ chứa máy chủ web và thêm thông số cookie vào mỗi dòng máy chủ để đặt giá trị của cookie
</br>
  - Khi đó sử dụng ‘cookie cookie_name insert \<options>’. “cookie_name” là giá trị mà HAProxy sẽ chèn vào (insert). Khi client quay lại (tức là cũng là client này và request tiếp theo), HAProxy sẽ biết được server nào để chọn cho client này. Ví dụ:

    ```sh
    cookie  WEB insert
    server web1 192.168.1.110:8080 cookie web1 check
    server web2 192.168.1.111:8080 cookie web2 check
    server web3 192.168.1.112:8080 cookie web3 check
    ```

</br>
  - Khi sử dụng insert, HAProxy phản hồi cho client là “WEB=web1”

***

- **Hạn chế:** Với sticky session, việc các request từ một user sẽ chỉ cố định vào một server. Vì vậy mà sẽ không đảm bảo được tính điều phối nhiều request từ một users đến nhiều server. Để khắc phục điểm hạn chế này, thì hiện nay có một số phần mềm như redis, memcached, … cho phép lưu session của user, còn việc điều phối các request của user thì vẫn thực hiện bình thường đến các server.

***

- **Health Check:**
  - HAProxy sử dụng health check để phát hiện các backend server sẵn sàng xử lý request. Kỹ thuật này sẽ tránh việc loại bỏ server khỏi backend thủ công khi backend server không sẵn sàng. health check sẽ cố gắng thiết lập kết nối TCP tới server để kiểm tra backend server có sẵn sàng xử lý request.
</br>
  - Nếu health check không thể kết nối tới server, nó sẽ tự động loại bỏ server khởi backend, các traffic tới sẽ không được forward tới server cho đến khi nó có thể thực hiện được health check. Nếu tất cả server thuộc backend đều xảy vấn đề, dịch vụ sẽ trở trên không khả dụ (trả lại status code 500) cho đến khi 1 server thuộc backend từ trạng thái không khả dụ chuyển sang trạng thái sẵn sàng.

***

- **HAProxy Runtime API:** được tích hợp vào HAProxy, cho phép thay đổi cấu hình HAProxy ngày lập tức, đồng thời HAProxy API cũng cho phép chiết xuất dữ liệu thông kê về HAProxy. Không giống như các API HTTP thông thường, HAProxy API chạy thông qua TCP và Unix sockets. Đó là lý do khi cấu hình HAProxy API, ta phải thiết lập socket mới cho HAProxy API (stats socket hoặc socket) và đôi khi trong một số tài liệu nói về HAProxy API lại sử dụng thuật ngữ ‘socket’ để thay thế.
