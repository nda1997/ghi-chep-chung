---
# Tìm hiểu load-balancer và haproxy
### 1. Load balancer

- Cân bằng tải giúp xử lí , phân bố lưu lượng truy cập giữa nhiều máy chủ có cùng chức năng trên hệ thống từ đó giảm thiểu thời gian chết của dịch vụ . 
- Bộ cân băng tải có thể phát hiện bất cứ máy chủ nào đang bị lỗi từ đó chuyển đều lưu lượng sang các máy chủ còn lại .
- Khi thực hiện cấu hình cân bằng tải nên chú ý 3 tiêu chí: 
  - Chọn địa chỉ IP và cổng sẽ nhận lưu lượng từ bên ngoài 
  - Xác định các máy chủ sẽ nhận lưu lượng từ bộ cân bằng tải
  - Đặt các quy tắc cụ thể cho từng yêu cầu

- Có 2 loại cân bằng tải :
  - Layer 4 : đơn giản là xử lí qua các giao thức mạng ( IP , TCP/UDP...) . Không có khả năng nhận biết nội dung dữ liệu , chỉ có khả năng hiện thị thông tin mạng là các cổng và địa chỉ ip 
  - Layer 7 : có thể truy cập dữ liệu ứng dụng và được ủy quyền để giải mã dữ liệu nhận được .
        ( Ví dụ như khi nhận được yêu cầu GET /picture thì LB sẽ tìm được máy chủ nào chịu trách nhiệm xử lí hình ảnh thì sẽ gửi tới nó )

- Các thuật toán sử dụng trong cân bàng tải
  - Round robin:
  - Weighted round robin
  - Least connections
  - Least response time
  -  IP Hash

### 2. Haproxy
#### 2.1. Giới thiệu
- HAProxy là phần mềm miễn phí , cung cấp cân bằng tải và proxy server .
#### 2.2. Tính năng
- TCP proxy : chấp nhận kết nối TCP từ 1 listening socket , kết nối tới server và gắn các socket này lại với nhau thông qua lưu lượng được gửi theo cả 2 hướng 
- HTTP reverse-proxy : nhận các yêu cầu http trên 1 listening TCP socket và chuyển các request này tới các server khác nhau bằng các kết nối khác nhau
- SSL terminator / initiator / offloader
- TCP normalizer
-  HTTP normalizer
-  HTTP fixing tool
-  content-based switch
-  server load balancer
-  traffic regulator
-  protection against DDoS and service abuse
-  observation point for network troubleshooting
-  HTTP compression offloader
-  caching proxy
-  FastCGI gateway

