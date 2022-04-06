# KVM
Tìm hiểu ảo hóa KVM
Phụ lục
1.	KVM là gì ?
2.	KVM hoạt động như thế nào
3.	Kiến trúc KVM
4.	Lab


1. Khái niệm
KVM (Kernel-base virtual machine ) là 1 module được tích hợp vào nhân linux từ bản phân phối 2.6.20
Công nghệ tạo là 1 môi trường trung gian để giao tiếp giữa phần cứng máy tính và phần mềm chạy trên nó
Nó được phát hành vào ngày 5 tháng 2 năm 2007 tại isarel
 Qemu là  hypervisor loại 2 : giả lập tài nguyen phần cứng ( disk , card mạng, cpu…) nhưng mapping từ vcpu - cpu kém
KVM là hypervisor loại 1 : mapping từ vcpu  cpu tốt nhưng giả lập và quản lý phần cứng kém


Kiến trúc KVM-QEMU
 
 ![image](https://user-images.githubusercontent.com/50499526/161735169-241c97d8-f899-4eda-ba46-97fff2f78848.png)

Thành phần trong KVM
Physical NIC : card mạng vật lí  của máy host
Host NIC driver : Hỗ trợ tương tác giữa hardware máy và phần mềm trong OS
Virtual bridge (Linux bridge ): chức năng tương tư như switch , hỗ trợ về mặt network trong máy ảo
KVM Kernel module : KVM được xây dựng trên nhân của linux
TAP:  1 port trên virtual bridge kết nối giữa switch mà port mạng của máy ảo
vNIC: (virtual nic ): port mạng ảo tương tự như pNIC được hỗ trợ bởi kvm


![image](https://user-images.githubusercontent.com/50499526/161735301-410fcf37-ba1d-445a-9372-ba168ebebc52.png)





Mô hình vận hành KVM
•	KVM giúp cung cấp một số thành phần cấp hệ điều hành : trình quản lý bộ nhớ, bộ lập lịch xử lý, ngăn xếp đầu vào / đầu ra (I / O), trình điều khiển thiết bị, trình quản lý bảo mật, ngăn xếp mạng … để có thể chạy ảo hóa.
•	Mọi ảo hóa sẽ được triển khai như một quy trình Linux thông thường, được lên lịch sẵn bởi bộ lập lịch Linux tiêu chuẩn,với phần cứng ảo chuyên dụng như card mạng, bộ điều hợp đồ họa, CPU, bộ nhớ và đĩa.
![image](https://user-images.githubusercontent.com/50499526/161735140-c1b853f6-ab34-40c3-ba1c-0ea24a781e51.png)

 

-	B1: KVM khởi chạy ở chế độ user mode gọi thư viện ioctl() để thực thi yêu cầu ( ví dụ như gửi khung ethernet ra ngoài internet , tạo ổ đĩa ảo ... )
-	B2 : chế độ Kernel-mode cung cấp thông tin tới phần cứng để thực hiện yêu cầu từ ioctl()
- B3 : chế độ guest-mode : ( thực hiện code ) nhận thông tin và cung cấp  phần cứng để thực hiện yêu cầu ảo hóa và gửi tín hiệu đến cho kernel để tiếp tục luồng xử lí



