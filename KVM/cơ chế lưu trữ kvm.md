## Mục lục

[1, Thick provisioning](#1)

[2, Thin Provisioning](#2)

Khi khởi tạo VM (Virtual Machine) có bước lựa chọn định dạng phân vùng lưu trữ cho VM như: Thin Provisioned, 
Thick Provisioned Lazy Zeroe & Thick Provisioned Eager Zeroed. Điểm khác biệt ở đây là cách sử dụng tài nguyên lưu trữ trên disk.

<a name="1"></a>
## 1. Thick provisioning


- Toàn bộ dung lượng lưu trữ đĩa ảo sẽ được phân bố cấp phát trước trên đĩa vật lí khi đĩa ảo được tạo.
- Đĩa ảo sẽ sử dụng được toàn bộ không gian lưu trữ đã được cấp do đó các máy ảo khác sẽ không sử dụng chung được với đĩa ảo này.
- Có 2 loại thickprovisioning :

  + Lazy zeroed disk: chiếm toàn bộ dung lượng tại thời điểm tạo , có thể chứa một số dữ liệu cũ trên đĩa  vật lý. Dữ liệu cũ này không bị xóa hoặc ghi đè lên, do đó, nó cần phải được "xóa sạch " trước khi dữ liệu mới có thể được ghi vào các khối. Loại disk này có thể được tạo nhanh hơn, nhưng hiệu suất của nó sẽ thấp hơn cho lần ghi đầu tiên do IOPS tăng cho các block mới.  ( Có thể so sánh tương đương với việc quick fomat)

  + Eager zeroed disk: cũng có toàn bộ không gian lưu trữ tại thời điểm tạo , không có dữ liệu cũ trên đĩa vật lí . Nó sẽ thực hiện việc ghi toàn bộ bit 0 lên tất cả các sector và sau đó ghi dữ liệu mới bằng cách điền bit 1 vào . ( Có thể so sánh tương đương với việc Full format)

![image](https://user-images.githubusercontent.com/50499526/165014350-4e99ed4f-df8b-4dc3-90ea-4210e36382cf.png)

- Như trên hình minh họa ta có thể thấy disk 1 và disk 2 được cấp mặc định 30GB . Trên datastore có thể thấy không còn dung lượng trống nào có thể sử dụng cho các VM khác mặc dù là 2 disk mới sử dụng hết có 10GB lưu trữ và vẫn còn dư 40GB .Như vậy các đĩa ảo đảm bảo sự độc lập nhưng lại lãng phí tài nguyên .
<a name="2"></a>
## 2. Thin Provisioning

![image](https://user-images.githubusercontent.com/50499526/165014889-ff2e3941-7d4c-4e90-ae0d-e2d242fe443b.png)


- Thin provisioning là một loại phân bổ trước bộ nhớ khác. Đĩa ảo được tạo kiểu thin provisioning chỉ tiêu thụ không gian cần thiết ban đầu và tăng theo thời gian theo nhu cầu.
- Như trên hình minh họa 2 đĩa ảo sử dụng thin được cấp 30 GB nhưng lại sử dụng hết 10GB lưu trữ tại thời điểm tạo . Như vậy trên datastore 60Gb cấp phát ta vẫn dư 40GB để có thể sử dụng cho máy ảo khác hoặc mở rộng disk khác .


# So sánh thick vs thin provisionning
![image](https://user-images.githubusercontent.com/50499526/165016611-8ebb8d89-de34-4082-b1e4-769a7bf61dbe.png)
