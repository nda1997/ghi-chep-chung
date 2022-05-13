# Kiến trúc tổng quan trong OpenVswitch
--- 
[1. Khái niệm chung](#2)

[2. Openvswitch Architechture](#1)

  [2.1. Mô hình](#5)
  
  [2.2.Thành phần chính](#3)
  
  [2.3.Các thức hoạt động](#4)


<a name='2' ></a>
### Khái niệm chung
- Openvswitch là 1 phần mềm  cung cấp môi trường ở lớp 2 và 3 trong mô hình OSI . Sử dụng nhiều ở điện toán đám mây nhắm quản trị , kết nối các máy ảo trong đó.
- OpenvSwitch (OVS) hỗ trợ nhiều nền tảng ảo hóa như Xen/XenServer, KVM, và VirtualBox.
- OVS hỗ trợ các tính năng sau: VLAN tagging & 802.1q trunking, spaning tree , LACP, SPAN/RSPAN, Tunneling Protocols ,QoS

<a name= '1'></a>
### Kiến trúc
---
- Mô hình 

![image](https://user-images.githubusercontent.com/50499526/167980294-3704de46-c152-4c4c-815c-563d990e90d6.png)

- Như trên hình ta có thể thấy có 3 lớp chính trong OVS :
  + Kernel space : lớp nhân tiếp nhận xử lí luồng tin
  + User-space :  người dùng tương tác với các trình quản lí trong OVS 
  + Remote: Lớp giám sát 
- Các thành phần  chính :

#### ovs-vswitchd : 
  - Đóng vai trò deamon thực hiền các chức năng chuyển mạch trong ovs . Nó giao tiếp với controller bằng giao thức openflow , với ovsdb-server bàng giao thức ovsdb , với datapath bằng giao thức netlink . Nó sẽ đọc cấu hình trong ovsdb-server khi khởi động sau đó sẽ tự cấu hình flow table .
  
  
  
#### ovsdb-server : 
  - Cơ sở dữ liệu của OVS , chứa cấu hình, giao diện,  VLAN, tạo chuyển mạch ảo, card mạng ...
 
  ` /etc/openvswitch/conf.db` :  là nơi lưu trữ cơ sở dữ liệu của ovsdb-server , được lưu dưới dạng json
  
  - Các kết nối socket được lưu tại /var/run/openvswitch/ 

![image](https://user-images.githubusercontent.com/50499526/168020627-47cfd3fb-e6af-4f56-9d84-ea68e58cc345.png)

Ví dụ như ta muốn quản lý datapath hoặc muốn truy xuất database thì các câu lệnh sẽ được chuyển qua các kết nối socket này .

 `/var/log/openvswitch/` : File lưu nhật kí của OVS 
 
 ![image](https://user-images.githubusercontent.com/50499526/168021484-123b7c70-7c48-4598-b043-8f5e9d77f832.png)


#### datapath kernel (openvswitch.ko)
  - nhận gói tin từ nhiều luồng , chuyển tiếp và thực thi các action trên gói tin .

![image](https://user-images.githubusercontent.com/50499526/168018182-349eae22-1b54-4f87-8c64-cfb2c7003cb1.png)
 
  - Ảnh trên là thông tin cơ bản của datapath kernel bao gồm :
  
  ` /lib/modules/3.10.0-1160.el7.x86_64/kernel/net/openvswitch/openvswitch.ko.xz` : vị trí của file 
  ` depends` : thư viện phụ thuộc
  ` chữ kí số ` 

- Flow : Luồng xử lí 1 gói tin ( gói tin đi vào port nào , có gắn vlan không ...)

![image](https://user-images.githubusercontent.com/50499526/168023244-5a230cf8-41ac-415c-941c-0aaa6228b7db.png)


Ta tiến hành kiểm tra 1 flow có trong ovs . Thông thường 1 flow sẽ có các tham số cơ bản sau 
  + duration : thời gian sống
  + table : lưu tại bảng dữ liệu nào
  + n_packets: số packet truyền trong mạng
  + n_bytes : số bytes truyền trong mạng
  + actions :  hành động của gói tin này là gì  ( chuyển tiếp bình thường, tới cổng nào , drop ...)

- Controller :  Điều khiển và chỉ cho datapath xử lí luồng gói tin .
- 
- Ngoài ra sẽ có các công cụ hỗ trợ :
  + ovs-dpctl : trình điều khiển có các câu lệnh tương tác với deamon để quản lý datapath ( nó cũng có thể tương tác trực tiếp với lớp datapath kernel mà không cần thông qua ovs-switchd)
  + ovs-appctl : tập hợp dòng lệnh gửi xuống ovs-switchd 
  + ovs-vsctl : truy vấn tới database ( thực hiện ghi , nhân, thay đổi chỉnh sửa cấu hình )
  + ovsdb-client : tương tác với ovsdb-server để 
  + ovsdb-tool : tương tác trực tiếp với database không qua ovsdb-server
  + ovsdb : chứa các table lưu cấu hình
  + ovs-ofctl : quản lý flow table, monitor ovs , cấu hình thông qua giao thức openFlow
  + sFlowTrend :  giám sát switch theo giao thức sflow
 


<a name= '4'></a>
### Mô tả luồng xử lí 1 gói tin

![image](https://user-images.githubusercontent.com/50499526/168194959-8590b63c-05cb-4a89-914b-416ba5ddde86.png)

1. Datapath nhận gói tin từ thiết bị ngoài 
2. Sẽ có 2 hướng xử lí gói tin trong datapath kernel :
  2.1. Datapath kiểm tra trong bộ nhớ đệm (cache) của mình xem có các flow tương ứng cho gói tin đó không . Nếu có nó sẽ thực thi action trên flow đó (gói tin sẽ được chuyển tiếp tới 1 port khác hoặc gắn vlan ....) . Thông thường cache sẽ lưu flow gần nhất mà datapath xử lí , các flow cũ hơn sẽ bị xóa bỏ.
  2.2. Datapath kiểm tra không thấy có trong cache , nó chuyển tiếp gói tin tới ovs-switchd ở user-space để xử lí tiếp
3. ovs-switchd tiếp nhận gói tin. Nó sẽ hỏi controller cách xử lí flow này như thế nào
4. ovs-switchd sẽ tương tác với ovsdb-server để truy cập tới database . Nó kiểm tra flow table về các flow đang xử lí và sẽ lưu dữ liệu để phục vụ cho các lần truy vấn sau
5. ovs-switchd sẽ gửi lại các gói tin vào datapath để xử lí cho đúng luồng
6. Datapath check flow table để xử lí flow cho đúng luồng
7. Datapath sẽ chuyển tiếp gói mạng đi hoặc loại bỏ theo các flow trong database


**Note** : Nếu như ovs-switchd kiểm tra trong  flow table mà vẫn không thấy dữ liệu cho việc xử lí flow đó như thế nào thì sẽ dẫn đến việc các flow chuyển đến datapath sẽ bị datapath drop-action. Như vậy sẽ xảy ra hiện tượng như Network unreachable , disconected ... 


## Cơ sở dữ liệu cho Open vSwitch (ovs-db) 

- Sử dụng câu lệnh `ovsdb-client dump` để lấy dữ liệu database từ tệp /etc/openvswitch/conf.db

![image](https://user-images.githubusercontent.com/50499526/168203774-bace1a18-9e0e-4b4d-b814-e89e0816304b.png)

![image](https://user-images.githubusercontent.com/50499526/168203791-25a202a7-5927-4a32-9eb8-652b7fa6ad45.png)

Ta sẽ nhận thấy các trường dữ liệu của openvswitch để quản lý flow và các bridge

- Mối quan hệ giữa các thành phần trong bảng dữ liệu

![image](https://user-images.githubusercontent.com/50499526/168205354-237c5ebe-915f-4bcd-873d-89b2e0a0f14e.png)



