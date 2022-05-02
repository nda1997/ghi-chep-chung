# LAB Live Migration

### Mục lục

[1. Mô hình ](#1)

[2. Cơ chế ](#2)

[3.1 Cold migrate](#4)

[3.2 Heat migrate ](#5)

<a name="1"></a>
## 1. Mô hình

![image](https://user-images.githubusercontent.com/50499526/166237341-fc839610-dc19-4cde-9612-3829cdfac72a.png)

- 2 máy cài KVM và  nfs-client  
- 1 máy cài nfs-server 
<a name="2"></a>
## 2.Cơ chế 
- KVM live migration cung cấp 2 loại :
   + cold migration (static migration)
   + heat transfer (dynamic transfer) 

- Static migrate : Là phương án thủ công để di chuyển VM qua lại giữa 2 host KVM. Phương pháp này không cần NFS-server 
  + Các bước thực hiện cũng đơn giản :
     + B1 : Xác định thư mục lưu ổ đĩa và file xml của VM đó trên host nguồn , thực hiện copy sang thư mục lưu trữ của host đích
     + B2 : Define lại máy ảo trên host mới

- Dynamic transfer : 2 máy host cần có 1 thư mục lưu trữ chung ( ở trường hợp này tôi sử dụng nfs-server làm trung gian ). Do ổ đĩa ảo đã được lưu trữ trên pool storage chung nên quá trình thực hiện chỉ cần gửi trạng thái vcpu , bộ nhớ ram , trạng thái VM đến KVM đích
   + Tiến trình migrate mô tả cụ thể như sau :
      + Khi chuyển dịch máy ảo vẫn đang chạy như bình thường , bộ nhớ ram và các tiến trình được copy sang host đích 
      + QEMU / KVM sẽ theo dõi và ghi lại bất kỳ thay đổi nào của tất cả các trang bộ nhớ đã được chuyển trong quá trình di chuyển và bắt đầu chuyển các thay đổi của các trang bộ nhớ trong quá trình trước đó sau khi tất cả các trang bộ nhớ đã được chuyển
      + QEMU / KVM sẽ ước tính tốc độ truyền trong quá trình di chuyển. Khi dữ liệu bộ nhớ còn lại có thể được truyền trong một khoảng thời gian đã định (mặc định là 30 ms), QEMU / KVM sẽ tắt máy khách trên máy chủ nguồn, sau đó chuyển dữ liệu còn lại đến máy chủ đích.
      + Xác định lại máy ảo
<a name="5"></a>
## 3.2 Cấu hình Heat migrate 
Tắt `firewalld` trên cả 3 máy

`systemctl stop firewalld`

Disable selinux trên 2 máy cài đặt KVM

`setenforce 0`

**Thiết lập kết nối qemu trên cả 2 máy cài KVM**

Trên cả 2 máy vào file `/etc/libvirt/libvirtd.conf` bỏ comment ở các dòng sau

```
listen_tls = 0
listen_tcp = 1
tcp_port = "16509"
listen_addr = "0.0.0.0"
```

Bỏ comment dòng `LIBVIRTD_ARGS=”--listen”` trong file `/etc/sysconfig/libvirtd`

Restart lại libvirtd trên cả 2 máy

```
systemctl restart libvirtd
```

**Thiết lập NFS**
- Cài đặt cho NFS server : 
```
[root@localhost ~]# systemctl stop firewalld
[root@localhost ~]# setenforce 0
//Experimental environment, simply shut down the firewall and SElinux
[root@localhost ~]# yum -y install nfs-utils rpcbind
//Install software required for NFS
[root@localhost ~]# mkdir /nfs-share / / create shared directory
[root@localhost ~]# echo "/nfs-share  172.16.1.0/24(rw,sync,no_root_squash)" >> /etc/exports
//Configure shared directory permissions, shared users
//No root squash: make it obtain root permission of NFS server
[root@localhost ~]# systemctl start rpcbind / / start the remote transmission control service
[root@localhost ~]# systemctl start nfs / / start the NFS shared service
```

- Kiểm tra trên 2 KVM xem thư mục đã được mount chưa 

![image](https://user-images.githubusercontent.com/50499526/166242565-a87ad804-8ee6-4043-96be-db27305fd580.png)

![image](https://user-images.githubusercontent.com/50499526/166242623-5a7403c8-e63f-4e5a-8428-e131dd319b90.png)

- Tiến hành tạo pool share trên 2 KVM

![image](https://user-images.githubusercontent.com/50499526/166243992-79187922-412d-4b46-ad13-6ef83d8e05f3.png)

![image](https://user-images.githubusercontent.com/50499526/166244406-52d90f81-6b29-4715-bb91-72d26673b6ca.png)

`target path` :  đường dẫn thư mục share file của KVM
`host name` : IP của nfs server
`source path` : đường dẫn thư mục share file của NFS-server

- Tiếp tục tạo volume để lưu disk của VM trong pool share

![image](https://user-images.githubusercontent.com/50499526/166245361-f3e3155b-ca56-437f-b625-6661a70fd90d.png)

`backing store` : ta có thể chọn thêm option này để ổ đĩa này chỉ ghi dữ liệu đè lên ổ đĩa kia

![image](https://user-images.githubusercontent.com/50499526/166246091-99ed0ae3-a2ec-4b77-979f-7385d376716d.png)

#### Làm tương tự trên host KVM còn lại

- Giờ tiến hành tạo 1 VM trên KVM2
 ##### Lưu ý chọn đúng volume vừa tạo làm ổ đĩa của VM 
 
![image](https://user-images.githubusercontent.com/50499526/166246423-1efabcbc-72ac-49fd-9a76-e6f5edd00739.png)

- Kết nối đến KVM2 

![image](https://user-images.githubusercontent.com/50499526/166247394-08a2c058-506f-4059-a166-26466e5b1eb0.png)

- Kết nối thành công và ta có thể nhìn thấy VM vừa tạo trên KVM còn lại 

![image](https://user-images.githubusercontent.com/50499526/166247503-115a94e7-0184-4beb-97de-604d23c16781.png)

- Heat migrate 
  + Kiểm tra trạng thái VM trước khi migrate

![image](https://user-images.githubusercontent.com/50499526/166248163-0499af95-f353-4976-82cc-1358bd6361a4.png)

![image](https://user-images.githubusercontent.com/50499526/166248274-798e6212-0da1-47f5-8bde-71e524be287a.png)

- Trạng thái của VM trong thời gian và sau khi migrate

![image](https://user-images.githubusercontent.com/50499526/166248462-fccc946c-6b2f-40bd-b025-f3487d0b3839.png)

![image](https://user-images.githubusercontent.com/50499526/166248609-b01ae086-acb5-403b-9836-67d4905adcc3.png)

![image](https://user-images.githubusercontent.com/50499526/166248755-cecc930b-2119-4a76-8b17-1f7b72d6988b.png)

- Sau khi migrate thành công ta define file XML của VM đó trên host mới để tránh việc reset bị mất VM
### Như vậy là heat migrate thành công


