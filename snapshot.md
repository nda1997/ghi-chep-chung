# Mục lục
## [1. Hướng dẫn tạo Snapshot](#1)

### [1.1  Internal Snapshot](#2)

### [1.2  External Snapshot](#3)

---
### <a name="1"> 1. Hướng dẫn tạo Snapshot trên VM </a>
- Snapshot là 1 bản lưu của hệ thống tại thời điểm ta tạo ra nó. Nó ghi lại toàn bộ dữ liệu , cài đặt , ứng dụng  trạng thái của máy ảo và ta có thể quay trở  lại thời điểm đó 1 cách dễ dàng.
- libvirt hỗ trợ tạo snapshot khi máy ảo đang chạy và có 2 loại snapshot được hỗ trợ trong kvm:
   + Internal 
   + External


#### <a name="2">  1.1  Internal Snapshot </a>
##### `Internal` : Khi tạo theo chế độ này thì các bản snapshot sẽ được lưu ngay trong ổ đĩa của VM
  + Ưu điểm : Rất dễ tạo
  + Nhược điểm : 
      - Sẽ bị giới hạn theo dung lượng disk của VM , tức là khi disk bị đầy ta không thể tạo thêm snapshot. Ta có thể pause VM lại rồi snapshot để không lưu trạng thái VM mà sẽ chỉ lưu dữ liệu lên disk từ đó sẽ đỡ lãng phí tài nguyên.
      - Chỉ hỗ trợ định dạng qcow2
      - Không hoạt động với LVM storage pools
      
- Một vài câu lệnh `virsh` liên quan tới việc tạo và quản lí máy ảo:
    - `snapshot-create` : Tạo snapshot từ file XML
    - `snapshot-create-as` : Tạo snapshot với những tùy chọn
    - `snapshot-current` : Thiết lập hoặc lấy thông tin của snapshot hiện tại
    - `snapshot-delete` : Xóa một snapshot
    - `snapshot-dumpxml` : Tạo ra thêm 1 file XML cho một snapshot
    - `snapshot-edit` : Chỉnh sửa file XML của snapshot
    - `snapshot-info` : Lấy thông tin của snapshot
    - `snapshot-list` : Lấy danh sách các snapshots
    - `snapshot-parent` : Lấy tên của snapshot "cha" của một snapshot nào đó
    - `snapshot-revert` : Quay trở về trạng thái khi tạo snapshot

- Ta tiến hành tạo snapshot theo lệnh sau 
    + virsh snapshot-create-as --domain VMname --name "snapshot1" --atomic
  ![image](https://user-images.githubusercontent.com/50499526/166205907-442c159f-2a36-4e5a-a6b4-64a9b7ccbe47.png)
 Trong đó option --atomic là để đảm bảo vẹn toàn dữ liệu

- Ta có thể kiểm tra thông tin snapshot vừa tạo 
![image](https://user-images.githubusercontent.com/50499526/166206717-c0ca0f43-be54-4d43-97e1-002d302268e4.png)
- Để quay trở lại trạng thái của một internal snapshot, dùng câu lệnh: `virsh snapshot-revert <vm-name> --snapshotname "Snapshot1"`
- Để xóa một internal snapshot sử dụng câu lệnh `virsh snapshot-delete VMname Snapshotname`
![image](https://user-images.githubusercontent.com/50499526/166206919-59cc7d7f-9837-4724-9121-b3df68d995b9.png)



#### <a name="3">  1.2  External Snapshot </a>

- External : Dựa trên cơ chế copy-on-write. Khi snapshot được tạo, ổ đĩa ban đầu sẽ có trạng thái "read-only" và có một ổ đĩa khác chồng lên để lưu dữ liệu mới

![image](https://user-images.githubusercontent.com/50499526/166216711-96e07828-de08-40ce-ae53-c105f47b16ee.png)

- Ổ đĩa được chồng lên được tạo ra có định dạng qcow2, hoàn toàn trống và nó có thể chứa lượng dữ diệu giống như ổ đĩa ban đầu. External snapshot có thể được tạo với bất kì định dạng ổ đĩa nào mà libvirt hỗ trợ. Tuy nhiên không có công cụ đồ họa nào hỗ trợ cho việc này.

**Tạo snapshot**
- Tiến hành kiểm tra ổ đĩa mà máy ảo muốn tạo snapshot đang sử dụng bằng câu lệnh `virsh domblklist VMname --details`
![image](https://user-images.githubusercontent.com/50499526/166207096-112cf321-0d7b-4200-bbb1-4650c90c836b.png)
- Tiến hành tạo snapshot bằng câu lệnh `virsh snapshot-create-as VMname snapshot1 --disk-only --atomic`
Trong đó `--disk-only` dùng để tạo snapshot cho riêng ổ đĩa.
![image](https://user-images.githubusercontent.com/50499526/166210195-8b2b3583-5471-46d1-8b86-a8a3b703c4bc.png)

- Kiểm tra lại tình trạng snapshot
![image](https://user-images.githubusercontent.com/50499526/166210363-31cd4e3b-56e3-44cc-a8e2-a5741e2fbd78.png)

- Kiểm tra lại ổ đĩa máy ảo 
- 
![image](https://user-images.githubusercontent.com/50499526/166210427-8526dce3-c7a0-4554-976a-3cf9cfb9e3b3.png)

Như ta thấy hiện giờ dữ liệu sẽ lưu vào đường dẫn mới là /var/lib/libvirt/images/anhnd-kvm02.snapshot1 thay vì là file qcow2 như ban đầu .
Ổ đĩa qcow2 sẽ chuyển trạng thái là read-only và trở thành backingfile cho ổ đĩa mới

![image](https://user-images.githubusercontent.com/50499526/166211322-e2007775-5b42-42a6-9dd1-fd69a0f9ae01.png)

**Revert lại trạng thái snapshot**
-  Do libvirt chưa hỗ trợ revert snapshot nên ta phải cấu hình file XML .

![image](https://user-images.githubusercontent.com/50499526/166211973-be3713d0-6b3e-4508-8cd2-0a70dab02414.png)

- Ví dụ ta có 2 bản snapshot là snapshot1 và snapshot2 . 'Snapshot2' là bản mới nhất và ta cần revert lại về bản snapshot1. Ta tiến hành như sau
    + Kiểm tra các bản snapshot 
    
![image](https://user-images.githubusercontent.com/50499526/166212431-ee873092-9bad-4009-a3e9-263e866458ad.png)

    + Lấy đường dẫn tới ổ đĩa được tạo ra khi snapshot:

![image](https://user-images.githubusercontent.com/50499526/166212552-e3c544f9-3fab-48f0-b5db-cdf63265ba4b.png)

    + Kiểm tra  backing file

![image](https://user-images.githubusercontent.com/50499526/166212755-3f8d8400-662a-4cd0-8a6a-61cf2597b664.png)

    + Chỉnh sửa  file XML,  thay thế ổ đĩa snapshot2 thành snapshot1

![image](https://user-images.githubusercontent.com/50499526/166213213-c7c67b60-6bb8-4b52-a29b-d753b77e8078.png)

    + Tiến hành define lại file XML và kiểm tra lại xem đã đúng đường dẫn chưa
    
![image](https://user-images.githubusercontent.com/50499526/166213427-b3a1cc79-d0c4-4bf7-a0ed-d6a26309575a.png)

    + Khởi động máy ảo và kiểm tra

#### Xóa external snapshot

- Ta tiến hành sửa file XML, thay thế ổ đĩa snapshot2 về qcow2

![image](https://user-images.githubusercontent.com/50499526/166216132-5dd96961-cc3e-4d5c-aa23-d47c0a0ccef2.png)

- Tiến hành xóa bỏ baseimages và metadata ( xóa bỏ phần định nghĩa backingfile trong xml )

![image](https://user-images.githubusercontent.com/50499526/166216369-5740ae75-35c2-46a9-aeb8-111dc79656b5.png)

![image](https://user-images.githubusercontent.com/50499526/166216516-5328d6bc-ea40-4546-bec3-4bd7a0689439.png)

- Khởi động lại máy ảo 









