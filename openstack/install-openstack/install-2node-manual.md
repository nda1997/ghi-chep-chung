# Ghi chép lại các bước cài đặt manual Openstack Train CentOS 7 môi trường Lab 

### Mục lục

[1. Mô hình triển khai](#mohinh)<br>
[2. IP Planning](#planning)<br>
[3. Thiết lập ban đầu](#thietlap)<br>
[4. Cài đặt node controller](#controller)<br>
[5. Cài đặt node compute](#compute)<br>
[6. Truy cập dashboard horizon](#dashboard)<br>

<a name="mohinh"></a>
## 1. Mô hình triển khai

Mô hình triển khai gồm 1 node Controller, 2 node Compute.

![image](https://user-images.githubusercontent.com/50499526/182786678-542b3c9d-a156-40b1-9959-cc2ed47d6d42.png)

Môi trường cài đặt những thứ cơ bản nhất .
Ở mô hình này các node kết nối , truyền dữ liệu  thông qua 1 đường mạng duy nhất
