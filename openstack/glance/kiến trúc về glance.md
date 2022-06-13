# Tìm hiểu tổng quan Image Service - Glance
### Mục lục
1. Giới thiệu về glance
2. Thành phần 
3. Kiến trúc 
4. Định dạng image trong glance

---

#### 1. Glance là gì

- Glance là thành phần chịu trách nhiệm về dịch vụ image trong openstack . Các chức năng của nó bao gồm tìm kiếm , đăng ký, thu thập thông tin về image. Bản thân glance không phải là nơi lưu trữ image .
- Giống như 1 proxy , glance liên kết với các thành phần khác trong openstack để người dùng tương tác và truy vấn đến các image cần thiết thông qua API mà nó cung cấp

#### 2. Thành phần chính 

![image](https://user-images.githubusercontent.com/50499526/173264015-42a77b4d-7d7a-412e-907d-929755c73d79.png)

- Glance gồm có các thành phần chính sau :
  + Glance-api : cung cấp REST API cho user về các lệnh gọi truy vấn, thu thập image
  + Glance-registry : thu thập , xử lí metadata của image ( ví dụ : kích thước và loại image )
  + glance-database: cung cấp nơi lưu trữ cho metadata 
  + store-backend: glance sẽ không tự lưu image , nó sẽ lưu ở vùng lưu phụ trợ ( swift, filesystem, S3...)

#### 3. Kiến trúc 

![image](https://user-images.githubusercontent.com/50499526/173264674-efb0ed4f-25dd-4b90-80bc-9e3d0b2115cd.png)

- Kiến trúc chính gồm các thành phần sau :
  + client :  user sử dụng rest-api để request với server 
  + REST API :  cung cấp các dịch vụ của glance thông qua REST
  + Glance Domain controller :  quản lí và thực hiện các chức năng như xác thực , cảnh báo, policies, quota...
  + Registry layer : giao thức hỗ trợ bảo mật cho tương tác giữu domain controller và database abstraction layer 
  + Database Abstraction Layer : chịu trách nhiệm kết nối giữa domain controller và database 
  + Store drivers : tổ chức tương tác giữa glance và các vùng lưu trữ image


#### 4. Định dạng image

**Disk Formats**

| Disk format | Notes |
|-------------|-------|
| Raw | Định dạng đĩa phi cấu trúc |
| VHD | Định dạng chung hỗ trợ bởi nhiều công nghệ ảo hóa trong OpenStack, ngoại trừ KVM |
| VMDK | Định dạng hỗ trợ bởi VMWare |
| qcow2 | Định dạng đĩa QEMU, định dạng mặc định hỗ trợ bởi KVM vfa QEMU, hỗ trợ các chức năng nâng cao |
| VDI | Định dạng ảnh đĩa ảo hỗ trợ bởi VirtualBox |
| ISO | 	Định dạng lưu trữ cho đĩa quang |
| AMI, ARI, AKI | Định dạng ảnh Amazon machine, ramdisk, kernel |

**Container Formats**

Container Formats mô tả định dạng files và chứa các thông tin metadata về máy ảo thực sự. Các định dạng container hỗ trợ bởi Glance

| Container format | Notes |
|------------------|-------|
| bare | Định dạng xác định không có container hoặc metadata đóng gói cho image |
| ovf | Định dạng container OVF |
| aki | Xác định lưu trữ trong Glance là Amazon kernel image |
| ari | Xác định lưu trữ trong Glance là Amazon ramdisk image |
| ami | Xác định lưu trữ trong Glance là Amazon machine image |
| ova | Xác định lưu trữ trong Glance là file lưu trữ OVA |
| docker | Xác định lưu trữ trong Glance và file lưu trữ Docker |

**Store**

- Glance hỗ trợ nhiều loại repository để lưu image :
  + file system : lưu trữ image trong hệ thống ( cụ thể trong đường dẫn : /var/lib/glance/images )
  + object store: hệ thống lưu trữ openstack swift , các image lưu theo định dạng object
  + block store : hệ thống lưu trữ openstack cinder, image lưu theo các khối dữ liệu
  + http : glance có thể đọc các iamge máy ảo có sẵn trên internet thông qua giao thức http 
  + rados block device (RBD) : lưu image vào cụm ceph.

### 5. Các file cấu hình của glance

- **glance-api.conf** : File cấu hình cho API của image service.
- **glance-registry.conf** : File cấu hình cho glance image registry - nơi lưu trữ metadata về các images.
- **glance-scrubber.conf** : Được dùng để dọn dẹp các image đã được xóa
- **policy.json** : Bổ sung truy cập kiểm soát áp dụng cho các image service. Trong này, chúng tra có thể xác định vai trò, chính sách, làm tăng tính bảo mật trong Glane OpenStack.

