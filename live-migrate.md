# LAB Live Migration

### Mục lục

[1. Mô hình ](#1)

[2. Cơ chế và cấu hình](#2)

[3. Live Migration](#3)
[3.1 Cold migrate](#4)
[3.2 Heat migrate ](#5)
<a name="1"></a>
## 1. Mô hình

![image](https://user-images.githubusercontent.com/50499526/166237341-fc839610-dc19-4cde-9612-3829cdfac72a.png)

- 2 máy cài KVM và  nfs-client  
- 1 máy cài nfs-server 

## 2. Cấu hình 
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

- Giờ tiến hành tạo 1 VM trên KVM1
 ##### Lưu ý chọn đúng volume vừa tạo làm ổ đĩa của VM 
 
![image](https://user-images.githubusercontent.com/50499526/166246423-1efabcbc-72ac-49fd-9a76-e6f5edd00739.png)



