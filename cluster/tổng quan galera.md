## Ghi chép tìm hiểu database mariadb

### Mục lục

[1. Tổng quan Maria Galera Cluster](#tongquan)<br>
[2. So sánh](#sosanh)<br>
[3. Thành phần](#thanhphan)<br>


<a name="tongquan"></a>
## 1. Tổng quan Maria Galera Cluster

- Galera có thể hiểu là 1 plugin thêm của mysql nhằm tạo ra nhiều bản sao giống nhau trên nhiều server khác nhau , phục vụ cho mục đích backup dữ liệu database .
- Galera hỗ trợ chế độ active-active tức là có thể truy cập , ghi dữ liệu đồng thời trên tất cả các node khác nhau.
- 

<a name="sosanh"></a>
## 2.Sự khác biệt giữa MySQL Replication và Galera Cluster

![image](https://user-images.githubusercontent.com/50499526/178454351-23567f18-ce16-4621-922a-4fc4e5ce1858.png)

- Khác với mô hình triển khai mysql replication  Các máy slave chỉ có thể đọc dữ liệu từ master sau khi ghi vào database . Ở đây galera coi các máy đồng thời đều là master và việc đồng bộ dữ liệu giữa các node là đồng thời thông qua wsrep-api

<a name="thanhphan"></a>
## 3. Thành phần chính 

- Galera gồm có 4 thành phần chính :
  + Database Management System : chạy trên từng node riêng biệt 
  + plugin: chịu trách nhiệm ghi dữ liệu qua lại giữa các node
  + wsrep api : xác nhận tương tác giữa các database của các node
  + Group Communication plugins : các plugin khác nhau của galera sẽ tương tác qua đây


## 4.Mô tả hoạt động 
