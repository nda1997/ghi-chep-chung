# Tìm hiểu một số mô hình mạng trong KVM
Cũng giống như các công cụ ảo hóa khác KVM cũng cung cấp các mô hình mạng trong việc ảo hóa network. Các mô hình bao gồm:
 * NAT
 * Bridge
 * Host-only
## 1. NAT
![image](https://user-images.githubusercontent.com/50499526/164135749-d3066988-7b0e-4593-85d6-87c79111e9c1.png)

Máy host KVM sẽ thực hiện ánh xạ 1 dải địa chỉ là 192.168.100.0/24 cho các VM sử dụng . VM bên trong có thể ra được ngoài internet nhưng từ bên ngoài sẽ không thấy được dải ip các VM sử dụng do đó không thể biết được VM nào bên trong mạng đó
Các bước để thực hiện tạo mạng NAT trong KVM :
- Mở trình quản lý virt-manager lên 
![image](https://user-images.githubusercontent.com/50499526/164136212-3aa31d6d-6dd9-4b86-ae2f-dc9e436c7448.png)
- Chọn Edit chọn Connection Details. Ta có thể thấy các mạng có ở cột bên trái. Để tạo một mạng khác ta làm như sau:
![image](https://user-images.githubusercontent.com/50499526/164136446-d14870f8-b778-43f7-ba51-c97947a4a432.png)
- Sau đó nhập tên cho mạng và nhấn Forward



- Ta chọn địa chỉ mạng cho mạng định tạo (chọn địa chỉ mạng tùy ý). Sau đó chọn dải IP cấp phát cho máy ảo trong dải mạng. Ta cũng có có thể để đặt IP tĩnh trên máy ảo nếu tích vào Enable Static Route Definition.( ở đây tôi không chọn ) Chọn xong ta nhấn Forward . 
![image](https://user-images.githubusercontent.com/50499526/164137679-894a3e3a-ef1d-421e-a768-6541a7cc6c33.png)

- Bước này ta có thể chọn có sử dụng IPv6 hay không. Ở đây tôi không chọn. Ấn Foward tiếp 

![image](https://user-images.githubusercontent.com/50499526/164139344-62d865d9-12f3-4b68-a7e0-0cca3b8f5d26.png)

- Ta chỉ ra mô hình mạng. Ở đây để là NAT sau đó chọn Finish để kết thúc.

![image](https://user-images.githubusercontent.com/50499526/164139839-a0017362-520a-4694-ac51-277bc68335c8.png)

- Sau khi tạo ta có thể thấy thông tin mạng NAT vừa tạo

![image](https://user-images.githubusercontent.com/50499526/164140489-78ff907d-7176-42ab-9d71-d10f235574a7.png)

Thao tác trên VM

![image](https://user-images.githubusercontent.com/50499526/164141573-56dd5ebf-660f-4156-9e18-458b59a23dc5.png)

- Ta vào Network source chọn đúng tên mạng chúng ta vừa tạo. Sau đó chọn Apply

![image](https://user-images.githubusercontent.com/50499526/164141998-8687ec90-6bc4-4446-8e6e-52217284857c.png)

- Ta reboot lại máy ảo và kiểm tra lại xem máy đã nhận đúng dải IP chưa

![image](https://user-images.githubusercontent.com/50499526/164142181-05ada4c1-75bc-4821-b9db-612bf0a05e32.png)

- Ở đây DHCP đã thực hiện cấp 1 ip đúng như trong dải như lúc ta tạo mạng . Ta tiến hành sửa file config port eth4 về đúng ip 192.168.200.100

![image](https://user-images.githubusercontent.com/50499526/164143173-79c3bc0f-6e24-49e0-9d7d-9310de024a97.png)

- Làm tương tự với VM2 

![image](https://user-images.githubusercontent.com/50499526/164144017-7eb01b6e-80fe-4712-8c96-78fa1af130d2.png)

- Tiến hành ping 8.8.8.8 và bắt gói tin trên máy host
![image](https://user-images.githubusercontent.com/50499526/164145146-8029357a-e2ef-48b1-b718-81b82c7f332e.png)

Gói tin đi qua bridge và được foward ra ngoài internet theo 1 port mạng của máy host (ta để chế độ any device ở bước số 4 nên máy host sẽ chọn ngẫu nhiên cho ta ).
Tiến hành từ máy tính khác ping vào địa chỉ máy VM1 không thể được

## 2. Host-only
Với mô hình mạng kiểu này cũng cho phép ta cấp phát địa chỉ tùy ý giống với mô hình NAT. Nhưng ở đây máy ảo không thể nói chuyện với máy tính bên ngoài. Nó chỉ có thể trao đổi với các máy trong cùng mạng bên trong server vật lý và trao đổi với đươc máy chủ vật lý.

### Cấu hình
- Ta cũng làm tương tự như cấu hình với NAT. Nhưng ở bước 4 ta chọn mục `Isolated virtual network` 

![image](https://user-images.githubusercontent.com/50499526/164147191-9feb3f27-cd09-4c62-95c7-11e328da45c6.png)

- Tạo mạng thành công như sau 
- 
![image](https://user-images.githubusercontent.com/50499526/164147734-9b163a31-11e9-4e75-a7e7-d59135b64c8b.png)

- Bây giờ tiến hành thao tác trên máy ảo. Ta chọng đúng tên mạng ta vừa tạo
- 
![image](https://user-images.githubusercontent.com/50499526/164147885-e2ed38b6-11cb-424a-8248-b639b3423f9f.png)

- Tiến hành reboot máy ảo và kiểm tra IP đồng thời ping ra ngoài internet
- 
![image](https://user-images.githubusercontent.com/50499526/164148260-9f5eda97-deba-40a6-ae3d-165510ff7fb8.png)

## 3. Bridge 
### 3.1. FLAT
- Mạng Flat hay còn gọi là mạng phẳng . Ở đây các VM trong cùng 1 host sẽ kết nối đồng cấp vs nhau .
- Mô hình :
![image](https://user-images.githubusercontent.com/50499526/164423526-0158795a-561d-44de-9de9-ca970a03b42b.png)
Các bước
### 3.2 Vlan
- Trên cùng 1 KVM

![image](https://user-images.githubusercontent.com/50499526/164579562-79597bb6-3199-4678-944e-9dc8f8225220.png)
Các bước thực hiện :
B1: Tạo 2 VM trên cùng 1 KVM , gắn port eth 2 máy vào cùng 1 vlan16 được dẫn từ sw core xuống máy host
B2: Ping 2 máy và bắt gói tin từ các phân đoạn để kiểm tra , cụ thể là vnet,eth, vlan16
B3: Kết luận

![image](https://user-images.githubusercontent.com/50499526/164579980-b5052dac-434a-49b5-acd5-558f949a2ae0.png)
![image](https://user-images.githubusercontent.com/50499526/164580000-2d028f46-c924-40f4-9065-6f557fe65410.png)
![image](https://user-images.githubusercontent.com/50499526/164580020-799016e2-4d3e-4239-8577-a330edeb9647.png)

Ở đây tôi tạo 2 VM có IP lần lượt là 172.16.1.22 và 172.16.1.29
Tiến hành ping và bắt gói tin
Trên eth1 , vnet , vlan16

![image](https://user-images.githubusercontent.com/50499526/164580326-9d781a96-7454-4d52-a14a-f6a423d3a646.png)

Kết luận : Ở đây gói tin chưa được đánh tag vlan id do nó đã được chuyển tiếp đến cùng 1 VM cùng dải ip , 2 Linux bridge đã chuyển tiếp dữ liệu cho nhau trên cùng 1 mạng FLAT

- Trên 2 KVM khác nhau cùng kết nối đến Switch

![image](https://user-images.githubusercontent.com/50499526/164576546-5305d2b4-803a-42e0-825a-1274747d990c.png)


Thành phần chính : 
- Switch được cấu hình vlan 16 và dẫn vlan xuống 2 host KVM .
- EM1 : ở đây là port mạng vật lí của 2 KVM

Các bước thực hiện
B1: Tạo 2 VM có trên đó có port mạng kết nối tới vlan 16 và đặt IP theo yêu cầu
B2: Ta tiến hành ping giữa 2 VM và bắt gói tin trên từng phân đoạn cụ thể là : eth,vnet,vlan16,em1 
B3: Kết luận đường đi của gói tin


![image](https://user-images.githubusercontent.com/50499526/164577339-eaa955a8-19cd-407d-83b6-775ff2493434.png)
![image](https://user-images.githubusercontent.com/50499526/164577374-a91f9032-3514-40f9-b096-959ac00c82c9.png)

Tiến hành ping và bắt gói tin trên các đoạn
kết quả thu tại eth(VM), vnet , vlan16

![image](https://user-images.githubusercontent.com/50499526/164577791-a0de156b-571e-49ff-b79a-cfcee406fd4a.png)

Kết quả thu gói tin tại card vật lí của máy host

![image](https://user-images.githubusercontent.com/50499526/164578116-53c62f2e-6f77-4b25-887d-559ec850592b.png)

Như ta thấy khi gói tin chuyển đến cổng vật lí của máy host nó sẽ được đánh tag vlan id vào rồi chuyển lên sw core để follow tiếp đến đúng port đã được cấu hình vlan16






