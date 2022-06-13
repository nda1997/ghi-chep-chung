---
### 1.Sử dụng glance cli

-  Hiển thị image được phép truy cập 

``` glance image-list ```

![image](https://user-images.githubusercontent.com/50499526/173268132-fbe59bec-db2a-4e7a-a8be-9e2f9d732796.png)

- Lấy thông tin về 1 image

``` glance image-show image_id ```

![image](https://user-images.githubusercontent.com/50499526/173268282-f687d296-1af5-4bca-bf7c-b303611e0429.png)

- Upload image từ file có sẵn lên glance 

![image](https://user-images.githubusercontent.com/50499526/173268928-e97edde8-450d-4982-b2f0-830ab64337af.png)

- tạo image rỗng và upload dữ liệu lên 

![image](https://user-images.githubusercontent.com/50499526/173269249-a25f9b26-654f-4779-8fad-444de237d34c.png)

- xóa image 

``` glance image-delete image_id ```

![image](https://user-images.githubusercontent.com/50499526/173269404-bb60e3ba-c56a-4b7b-a09f-107bffea670e.png)

### 2. sử dụng openstack client

| Glance CLI | OpenStack client | Mục đích |
|------------|------------------|----------|
| glance image-create | openstack image create | Tạo mới image |
| glance image-delete | openstack image delete | Xóa image |
| glance member-create | openstack image add project | Gán project với image |
| glance member-delete | openstack image remove project | Xóa bỏ image khỏi project |
| glance image-list | openstack image list | Xem danh sách image |
| glance image-download | openstack image save | Lưu image vào ổ đĩa |
| glance image-show | openstack image show | Hiển thị thông tin chi tiết của image |
| glance image-deactivate | openstack image set -deactivate | deactivate image |
| glance image-update | openstack set | Thay đổi thông tin image |


