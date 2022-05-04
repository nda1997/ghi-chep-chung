# Network trong KVM

[1. Tổng quan](#1)

[2. Kiến trúc](#2)

[3.Thành phần](#3)

[4. Hoạt động](#4)

<a name = "1"></a>
### 1.	Tổng quan
-	Brigde là cách kết nôi 2 Ethernet segmet với nhau thông qua giao thức độc lập. Các Packet được chuyển tiếp dựa trên Ethenet address ( cụ thể là MAC address ) , khác với IP address (như router)
-	Bridged network sử dụng để chia sẻ mạng của thiết bị thật tới các Vm (VMs). Mỗi VM được cung cấp địa chỉ có sẵn của router tại dải mạng thực cung cấp cho host. Bridged networking cho phép VMs kết nối với mạng bên ngoài thông qua card mạng vật lý máy chủ.
Note :  Không hỗ trợ kết nối wireless

<a name = "2"></a>
### 2.	Kiến trúc
![image](https://user-images.githubusercontent.com/50499526/161716957-9a0f0c6f-0edc-4e5c-8e1c-d44d340a2eaf.png)
 ![image](https://user-images.githubusercontent.com/50499526/161716992-da8c96a6-1910-4242-9c04-ffbedfd287db.png)

 
Note : các khái niệm cơ bản 
•  Virtual switch port : là 1 port ảo trên switch ảo . 
•  vNIC ( virtual network interface card ): đóng vai trò là card mạng cho máy ảo 
• Uplink port : nhận toàn bộ lưu lương mạng ra/vào từ switch đến mạng bên ngoài
• Virtual Ethernet port
#### 2.1	Kiến trúc Linux bridge
 ![image](https://user-images.githubusercontent.com/50499526/161717025-73de0cd1-c4f3-4f44-8815-e10534d5aef5.png)

<a name = "3"></a>
### 3. Các thành phần :

- Eth0: cổng mạng máy host
- Tap0: interface kết nối giữa switch ảo đến các VM
- Bridge : hoạt động tương tụ như switch ảo layer 2
- Vfs ( virtual file system ): phân vùng ảo để chứa file , dữ liệu 
- Fd ( forward database ): giao tiếp nhận dữ liệu giữa máy ảo và bridge 
- Read/write : dữ liệu được chuyển tiếp từ switch đến vfs

**Note** : VFS , FD có thể tìm thấy trong thư mục /proc  của máy host 
##### Các tính năng chính : 
- STP: Spanning Tree Protocol – giao thức chống lặp gói tin trong mạng
- VLAN: chia switch (do linux bridge tạo ra) thành các mạng LAN ảo, cô lập traffic giữa các VM trên các VLAN khác nhau của cùng một switch.
- FDB (forwarding database): chuyển tiếp các gói tin theo database để nâng cao hiệu năng switch. Database lưu các địa chỉ MAC mà nó học được. Khi gói tin Ethernet đến, bridge sẽ tìm kiếm trong database có chứa MAC address không. Nếu không, nó sẽ gửi gói tin đến tất cả các cổng.

TAP interface : 
-	Trong ảo hóa port mạng của VM chỉ xử lí được các frame Ethernet khác với vNIC xử lí khung ethernet . Nó sẽ bóc lớp header và chỉ chuyển tiếp lớp payload tới OS. 
Cấu trúc khung Ethernet
 ![image](https://user-images.githubusercontent.com/50499526/161717060-912b0dde-0ba2-4847-b8bc-dccb08c48318.png)

-	Do đó tap interface sẽ hỗ trợ chuyển tiếp khung Ethernet vào máy ảo để xử lí được như 1 port mạng vật lí
<a name = "4"></a>
### 4.	Hoạt động
Khi có  gói tin cần ra ngoài internet :
-  Gói tin sẽ được chuyển qua cổng eth0 của VM
-  Đi đến forward database từ cổng eth0
-  Từ FD sẽ tiếp tục đến vfs
-  Tại đây kernel sẽ lấy gói tin trong vfs chuyển đến bridge qua tap interface
-  Bridge sẽ lấy gói tin chuyển ra internet qua cổng eth0 của máy host

Chú ý đến các MAC address bôi đỏ như hình
![image](https://user-images.githubusercontent.com/50499526/163908107-1f6fa3f0-e1ce-4620-82fe-a2eded28f76e.png)
![image](https://user-images.githubusercontent.com/50499526/163908221-4034f32e-7054-49c3-90aa-8c835b3678e1.png)
Ở đây ta có thể thấy sw ảo virbr2 sinh ra 1 card mạng ảo là virbr2-nic đóng vai trò interface giao tiêp với các VM  nên MAC 2 cái này trùng nhau. Vnet0 là card mạng eth0 của VM cũng có mac trùng nhau nên có thể hiểu vnet0 là tap-interface .
Tiến hành ping và bắt bản tin lần lượt từ port eth0, vnet, virbr để kiểm tra luồng di chuyển của gói tin
- Trên eth0 của VM
![image](https://user-images.githubusercontent.com/50499526/163908657-d0805ba6-d242-4a14-800d-84817426c42f.png)
- Trên vnet 
![image](https://user-images.githubusercontent.com/50499526/163908713-2e8726e6-8c12-42b5-9f15-5dd2fbf8778e.png)
- Trên virbr2
![image](https://user-images.githubusercontent.com/50499526/163908764-2edf1cbc-fab6-47af-be3b-746e5beaf66c.png)

Như ta thấy gói tin chỉ đi qua 2 địa chỉ vật lí là 52:54:00:40:07:7d-card eth0 của VM và 52:54:00:6b:1a:f7- card sw ảo của máy host .toàn bộ các thành phần như tap, fd , vfs không nhìn thấy .

