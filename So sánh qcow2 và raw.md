# Định dạng ổ đĩa qcow2 và raw trong KVM
- Thông thường 1 ổ đĩa ảo của VM được lưu trong KVM có định dạng là qcow2 và raw . 2 chuẩn này là đại diện cho 2 cơ chế lưu trữ thin provision và thick provisioning 

## Kiểm tra dung lượng lưu trữ của 2 chuẩn 
- Tiến hành tạo ra 2 file có đuôi là qcow2 và raw rồi so sánh 

![image](https://user-images.githubusercontent.com/50499526/166505660-60c3dc9f-a444-4971-a48e-9d1a817c72b3.png)

![image](https://user-images.githubusercontent.com/50499526/166505843-8f18f78d-5338-430c-80ac-c9ea38067609.png)

Như ta thấy sau khi tạo ra 2 file có cùng dung lương là 10G nhưng khi kiểm tra trên hệ thống thì file raw thể hiện đúng 10G còn file qcow2 chỉ hiện có 193K dữ liệu

## Test hiệu năng đọc ghi

- Ta sử dụng lệnh dd để ghi dữ liệu vào trong và ra ngoài từ 2 ổ đĩa . ( ở đây các bạn có thể sử dụng tool trên tocdo.net để test cũng được )

![image](https://user-images.githubusercontent.com/50499526/166507363-37716b3c-c592-4b45-b9c9-423be1c236b1.png)

Như ta thấy thì kết quả có vẻ tương đối và hiệu năng của qcow2 nhỉnh hơn 1 chút. Về mặt lí thuyết thì cơ chế thick provisioning tốt hơn thin provisioning , tuy nhiên trong môi trường KVM thì hiệu năng 2 định dạng gần tương đương nhau khó có thể quyết định được 

## Chuyển đổi giữa raw và qcow2 
 
- Định dạng qcow2 được hỗ trợ nhiều hơn raw trong KVM . Vì vậy muốn chuyển đổi qua lại giữa 2 định dạng để phù hợp với yêu cầu thực tiễn ta dùng lệnh sau

``` qemu-img convert -f qcow2 -O raw file-qcow2 tên-file-mới ```

     option -f : ở đây là chọn fomart ổ đĩa 
     option -O : chọn format output

![image](https://user-images.githubusercontent.com/50499526/166510027-36240914-6ad2-4c76-9838-4a8c4680d519.png)

![image](https://user-images.githubusercontent.com/50499526/166510158-50ab2e1e-4659-4046-98d5-21f593016c18.png)

- sau đó ta bắt buộc phải sửa file xml và định dạng lại máy ảo

![image](https://user-images.githubusercontent.com/50499526/166510308-c0b418e5-5cb5-4743-b42a-de5e463673fe.png)

Sửa đuôi qcow2 thành raw và dùng lệnh `virsh define tên_file_máy_ảo.xml`


