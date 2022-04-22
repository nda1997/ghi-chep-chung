# KVM
Tìm hiểu ảo hóa KVM
Phụ lục
1.	KVM là gì ?
2.	KVM hoạt động như thế nào
3.	Kiến trúc KVM

[4. Quản lý KVM bằng libvirt](#3)

- [4.1. Giới thiệu](#3.1)

- [4.2. Các chức năng chính](#3.2)

- [4.3.Một số vấn đề cần biết với libvirt](#3.3)

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

# 4. Quản lý KVM bằng libvirt

<a name = "4.1"></a>
## 4.1. Giới thiệu

- **Libvirt** là một bộ các phần mềm mà cung cấp các cách thuận tiện để quản lý máy ảo và các chức năng của ảo hóa, như là chức năng quản lý lưu trữ và giao diện mạng. Những phần mềm này bao gồm một thư viện API, daemon (libvirtd) và các gói tiện tích giao diện dòng lệnh (virsh).

- Mục đích chính của Libvirt là cung cấp một cách duy nhất để quản lý ảo hóa từ các nhà cung cấp và các loại hypervisor khác nhau. Ví dụ, dòng lệnh `virsh list –all` có thể được sử dụng để liệt kê ra các máy ảo đang tồn tại cho một số hypervisor được hỗ trợ (KVM, Xen, Vmware ESX, … ). Không cần thiết phải học một tool xác định cho từng hypervisor.

<a name = "4.2"></a>
## 4.2. Các chức năng chính

-	**VM management – Quản lý máy ảo**: Quản lý vòng đời các domain như là start, stop, pause, save, restore và migrate. Các hoạt động hotplug cho nhiều loại thiết bị bao gồm disk và network interfaces, memory, và cpus.

-	**Remote machine support**: Tất cả các chức năng của libvirt có thể được truy cập trên nhiều máy chạy libvirt deamon, bao gồm cả các remote machine. Hỗ trợ kết nối từ xa, với cách đơn giản nhất là dùng SSH – không yêu cầu cấu hình thêm gì thêm. Nếu example.com đang chạy libvirtd và truy cập SSH được cho phép, câu lệnh sau sẽ cung cấp khả năng truy cập tới tất cả câu lệnh virsh trên remote host cho các máy ảo qemu/kvm:

  `virsh --connect qemu+ssh://root@example.com/system`

   Tham khảo thêm tại: https://libvirt.org/remote.html 

-	**Storage management**: bất kì host nào đang chạy libvirt daemon có thể được sử dụng để quản lý nhiều loại storage: tạo file image với nhiều định dạng phong phú (qcow2, vmdk, raw, …), mount NFS shares, liệt kê các nhóm phân vùng LVM, tạo nhóm phân cùng LVM mới, phân vùng ổ cứng, mount iCSI shares, và nhiều hơn nữa. vì libvirt làm tốt việc truy cập từ xa nên những tùy chọn này là có sẵn trên remote host.  (Xem thêm tại : http://libvirt.org/storage.html )

-	**Network interface management:** bất kì host nào chạy libvirt daemon có thể được sử dụng để quản lý các interface netowork vật lý và logic. Liệt kê các interface đang tồn tại, cũng như là cấu hình (hoặc tạo, xóa) các interfaces, bridge, vlans, và bond devices. 

-	**Virtual NAT and Route based networking:** Quản lý và tạo các mạng ảo, Libvirt virtual network sử dụng firewall để hoạt động như là router, cung cấp các máy ảo trong suốt truy cập tới mạng của host. (xem thêm tại: http://libvirt.org/archnetwork.html )

<a name = "4.3"></a>
## 4.3. Một số vấn đề cần biết với libvirt

Libvirt được cấu hình để lưu trữ các máy ảo QEMU và các file cấu hình XML trong thư mục `/etc`, nhưng việc sửa những file này thì không phải là cách để thay đổi thông tin cấu hình. Nếu sửa những file này và khởi động lại libvirtd có thể làm việc trong vài lần, có thể libvirtd sẽ ghi đè những thay đổi này và chúng sẽ bị mất. Quan trọng là nên dùng các công cụ của virsh hoặc các API khác để sửa file XML cho phép libvirt xác nhận những thay đổi của bạn. 

Sau đây là một số câu lệnh kết hợp với virsh áp dụng cho tất cả các loại XML của libvirt: 

\-	Virtual Networks: net-edit, net-dumpxml, net-start, …

\-	Storage Pools: pool-edit, pool-dumpxml, pool-define, ….

\-	Storage Volumes: vol-edit, vol-dumpxml, vol-define, ….

\-	Interfaces: iface-edit, iface-dumpxml, iface-start, ….
