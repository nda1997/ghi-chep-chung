Mô hình triển khai NFS

![image](https://user-images.githubusercontent.com/50499526/162895899-e33508df-48d7-47b7-b716-e8642df0d4ad.png)


*Note:  Ở đây sử dụng 2 máy centos 7.9*
# Cài đặt trên máy NFS server
### Cài đặt NFS
- Chạy lệnh :
    ``yum install nfs-utils``
### Tạo thư mục để share data giữa 2 máy host 
- mkdir -p  /root/share-data/ 
- chmod -R 755 /root/share-data/
- chown nfsnobody:nfsnobody /root/share-data/
### Chạy các dịch vụ cần thiết 
- systemctl start rpcbind
- systemctl start nfs-server
- systemctl start nfs-lock
- systemctl start nfs-idmap
