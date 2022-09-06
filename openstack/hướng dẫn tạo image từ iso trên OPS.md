# Hướng dẫn tạo máy ảo từ file ISO
---
### Ngữ cảnh đặt ra 
- Tạo máy ảo trực tiếp từ file iso khi không có image đóng gói sẵn trong kho lưu trữ 


1. Cài đặt image 

- Trên controller thực hiện tải file iso centos 7
```
wget http://centos-hcm.viettelidc.com.vn/7/isos/x86_64/CentOS-7-x86_64-Minimal-2009.iso 
```
- Upload lên glance
```
glance image-create --name C7 --disk-format iso --container-format bare --file /root/CentOS-7-x86_64-Minimal-2009.iso --visibility=public --property hw_qemu_guest_agent=yes --property os_type=linux --progress
```
2. Truy cập dashboard và tạo máy ảo từ file iso 

![image](https://user-images.githubusercontent.com/50499526/188574654-d43cb3d7-1954-4ef8-9072-256d941a08d2.png)

![image](https://user-images.githubusercontent.com/50499526/188574841-ff07539e-5f0e-4451-941c-b7ffd484d5de.png)

![image](https://user-images.githubusercontent.com/50499526/188575104-b89eb2b0-9e42-4936-b1a6-a19ea0bedecc.png)

- Đợi đến khi máy ảo running ta tiến hành tạo 1 ổ cứng mới và attachment vào máy ảo vừa tạo

![image](https://user-images.githubusercontent.com/50499526/188575549-1faff642-8aeb-47cc-9f39-85cad57f6cc2.png)

![image](https://user-images.githubusercontent.com/50499526/188575765-77cf2575-9029-4cd8-8d62-5763a19f8f8e.png)

- Ở đây ta tiến hành quét lại ổ đĩa 

![image](https://user-images.githubusercontent.com/50499526/188576044-b25a264d-b2d8-4321-98fc-c310aa9c3912.png)

![image](https://user-images.githubusercontent.com/50499526/188576113-b1193f7a-18d9-4137-a4f8-1967247cf3d8.png)

- Cài đặt và reboot lại máy 
- Lúc này máy ảo sẽ khởi động lại đĩa iso 1 lần nữa . Tắt và xóa máy ảo đi 
3. Tạo image từ volume 

- Từ mục volume chọn upload volume to image

![image](https://user-images.githubusercontent.com/50499526/188577239-987e5c45-ab73-4151-94d1-0d790cf815ad.png)

![image](https://user-images.githubusercontent.com/50499526/188577325-0198d5b6-29dc-4796-8efd-ec2a6a06acdb.png)

- Upload metadata cho image 

![image](https://user-images.githubusercontent.com/50499526/188577439-aa4a27e7-0cf5-4499-9873-a3fef2833fcb.png)

- ta thêm 3 mục sau 
```
hypervisor_type : kvm
auto_disk_config : true
hw_qemu_guest_agent : true
```
- Như vậy là image đã sẵn sàng để chạy máy ảo
- Ta có thể xóa volume ban đầu 
 
