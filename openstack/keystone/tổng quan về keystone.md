# Tìm hiểu tổng quan về keystone

## Mục lục

[1. Giới thiệu tổng quan về chức năng và lợi ích khi sử dụng keystone](#overview)

[2. Keystone Concepts](#concepts)

[3. Kiến trúc ](#identity2)

------

### 1. Tổng quan keystone

- Keystone là tên dự án của openstack identity service , một thành phần chịu trách nhiệm quản lí và ủy quyền danh tính. 
- Các chức năng chính là : xác thực người dùng , quản lý truy cập và bảo mật các kết nối 

**3 chức năng chính của keystone**

#### 1.1. Identity
- Nhận diện người đang cố truy cập vào tài nguyên cloud 
- Thông tin về user sẽ được lưu trữ trong database
- Nó cung cấp 1 số dịch vụ actor để lưu trữ và xác thực như  : SQL , LDAP, phần mềm bên thứ 3 ...

