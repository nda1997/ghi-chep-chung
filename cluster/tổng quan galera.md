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

![image](https://user-images.githubusercontent.com/50499526/178629498-7599cfce-9ca4-4218-b50d-80df0daaa83e.png)
- Người dùng commnit dữ liệu vào 1 node bất kì . 
- Ở đây mỗi sự thay đổi về dữ liệu được coi là 1 transation và sẽ được đánh số thứ tự .
- 1 node bất kì sẽ tiếp nhận hoạt động ghi dữ liệu , nó sẽ hỏi các node khác là có được phép không . 
  + Nếu xác thực thành công các thay đổi sẽ được thu thập và ghi vào 1 writeset . Writeset này sau đó sẽ được sao chép sang các node còn lại
  + Nếu xác thực không thành công thì các transation sẽ bị loại bỏ và transation ban đầu sẽ được khôi phục   
  `` quá trình này nhằm đảm bảo không có xung đột giữa các transation ``
- tiếp đó dữ liệu được commit sẽ được ghi vào log ( cache galera ) trên node đang thao tác
- wsrep (writeset-replicate ) chạy và ghi dữ liệu vào các node khác
- các node còn lại sẽ apply log 
- quá trình sao chép dữ liệu hoàn tất

***--- Trong trường hợp có nút bị lỗi ---***
 - Các nút khác vẫn tiếp tục đồng bộ dữ liệu .
 - Nút bị lỗi ( joiner ) sẽ request data từ cluster .
 - 1 nút khác đóng vai trò donor sẽ cung cấp data cho joiner thông qua phương thức State Snapshot Transfer (SST) - wsrep_sst_donor hoặc Incremental State Transfer (IST) 

Ta có thể tùy chọn phương thức cho cluster và trạng thái cho các node

  ```  wsrep_sst_method = rsync ```
  
  ```  wsrep_sst_donor  = "node1, node2" ```
  
    
    
