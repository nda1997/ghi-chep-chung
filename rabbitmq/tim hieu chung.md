### Tìm hiểu chung về RabbitmQ
---

1. rabbitmq là gì
2. Kiến trúc tổng quan 

![image](https://user-images.githubusercontent.com/50499526/190965540-646bc8e2-a9eb-43f4-ab51-0040f3783da5.png)

Giải thích các tham số
- Producer: người gửi thông điệp
- Consumer: nơi tiếp nhận , xử lí thông điệp
- Queue: bộ đệm lưu tin nhắn
- Connection: Kết nối TCP giữa Producer với RAbbitmq
- Chaanel : Kết nối ảo giữa 2 ứng dụng , các message sẽ chuyển qua đây
- Exchange: nhận messages từ producer và đẩy tiếp đến hàng đợi thông qua các binding
- Binding: liên kết giữa hàng đợi và Exchange
- Routing key : Khóa cung cấp địa chỉ cho messages, sẽ quyết định đường đi của messages
- AMQP (Advanced Message Queuing Protocol): giao thức chính của RAbbitmq
- Vhosts, virtualhost: Phân cho các ứng dụng khác nhau sử dụng chung 1 Rabbitmq, nhưng ko xung đột vào hàng đợi của nhau

![image](https://user-images.githubusercontent.com/50499526/190965613-7d02869f-5d50-4664-846b-257a1baaa820.png)

Luồng tin nhắn trong rabbitmq
B1: Producer gửi 1 tin nhắn qua kết nối TCP muốn tương tác với 1 ứng dụng nào đó
B1: The Exchange nhận tin nhắn và chịu trách nhiệm định tuyến tin nhắn đó đi đâu
    - Việc quyết định tin nhắn đi đâu dựa vào thuộc tính ( khóa định tuyến , type ...)
B3: Binding liên kết giữa exchange và queues  . Exchange dựa vào binding để quyết định gói tin đi đến hàng đợi nào
B4: Message sẽ ở trong hàng đợi cho đến khi nó đợi tiếp nhận bởi Consumer
B5: Consumer sẽ xử lí gói tin

###### Các loại Exchange
- Direct: Messages sẽ chuyển đến hàng đợi có binding key khớp với  routing key của messages 

![image](https://user-images.githubusercontent.com/50499526/190981236-6eda162d-5240-4d3c-97ba-b6acb352c997.png)
    
        - QueueA  ( create_pdf_queue) có liên kết với exchange với binding-key= pdf-create
        - Các message mới được chuyển đến có routing-key là pdf-create sẽ đc exchange định tuyến tới hàng đợi nào có binding-key=routing-key. Ở đây là queueA

- Fanout: Định tuyến toàn bộ message đến tất cả các hàng đợi có binding 

![image](https://user-images.githubusercontent.com/50499526/190981836-13127c18-67ee-4910-ac7d-953d04d0955e.png)

        - Message được chuyển đến tất cả các hàng đợi bất kể routing-key là gì 
- Topic: Thực hiện khi routing key khớp với routing pattern specified ở phần binding

![image](https://user-images.githubusercontent.com/50499526/190981394-87b14bc2-c108-40c3-a175-499b3129f8c6.png)

        - exchange nhận các message agreements
        - Kiểm tra thấy có 2 binding phù hợp là agreement-eu-berlin và agreement-eu-berlin-headstore
        - Routing tới 2 binding có các giá trị pattern khớp với exchange

- Headers: sử dụng thuộc tính tiêu đề cho việc định tuyến

![image](https://user-images.githubusercontent.com/50499526/190981881-8710da62-a730-48c6-affe-9cc444c1c7f1.png)

        - Message sẽ được chuyển tới queue có các trường ( key=value) khớp với binding
