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
![image](https://user-images.githubusercontent.com/50499526/164147734-9b163a31-11e9-4e75-a7e7-d59135b64c8b.png)

- Bây giờ tiến hành thao tác trên máy ảo. Ta chọng đúng tên mạng ta vừa tạo
![image](https://user-images.githubusercontent.com/50499526/164147885-e2ed38b6-11cb-424a-8248-b639b3423f9f.png)

- Tiến hành reboot máy ảo và kiểm tra IP đồng thời ping ra ngoài internet
![image](https://user-images.githubusercontent.com/50499526/164148260-9f5eda97-deba-40a6-ae3d-165510ff7fb8.png)





