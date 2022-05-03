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
