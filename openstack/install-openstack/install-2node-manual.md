# Ghi chép lại các bước cài đặt manual Openstack Train CentOS 7 môi trường Lab 

### Mục lục

[1. Mô hình triển khai](#mohinh)<br>
[2. IP Planning](#planning)<br>
[3. Thiết lập ban đầu](#thietlap)<br>
[4. Cài đặt node controller](#controller)<br>
[5. Cài đặt node compute](#compute)<br>
[6. Truy cập dashboard horizon](#dashboard)<br>

<a name="mohinh"></a>
## 1. Mô hình triển khai

Mô hình triển khai gồm 1 node Controller, 2 node Compute.

![image](https://user-images.githubusercontent.com/50499526/182786678-542b3c9d-a156-40b1-9959-cc2ed47d6d42.png)

Môi trường cài đặt những thứ cơ bản nhất .
Ở mô hình này các node kết nối , truyền dữ liệu  thông qua 1 đường mạng duy nhất

<a name="thietlap"></a>
## 3. Thiết lập ban đầu

- Trước tiên cài đặt 1 số dịch vụ cơ bản trên tất cả các node 

- Tắt selinux , firewalld
``` 
systemctl stop firewalld 
systemctl disable firewalld 
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

- Cấu hình mode sysctl

``` 
echo 'net.ipv4.conf.all.arp_ignore = 1'  >> /etc/sysctl.conf
echo 'net.ipv4.conf.all.arp_announce = 2'  >> /etc/sysctl.conf
echo 'net.ipv4.conf.all.rp_filter = 2'  >> /etc/sysctl.conf
echo 'net.netfilter.nf_conntrack_tcp_be_liberal = 1'  >> /etc/sysctl.conf
cat << EOF >> /etc/sysctl.conf
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.tcp_keepalive_time = 6
net.ipv4.tcp_keepalive_intvl = 3
net.ipv4.tcp_keepalive_probes = 6
net.ipv4.ip_forward = 1
net.ipv4.conf.all.rp_filter = 0
net.ipv4.conf.default.rp_filter = 0
EOF

```
Kiểm tra lại 
``` sysctl -p ```

#### Nếu báo lỗi thì là do thiếu module dưới kernel . thực hiện thêm 2 module sau 
```
modprobe br_netfilter
modprobe 

- Khai báo file host các node 
``` 
echo "172.16.4.200 controller" >> /etc/hosts
echo "172.16.4.201 compute01" >> /etc/hosts
echo "172.16.4.202 compute02" >> /etc/hosts
```

- Khai báo repo 
```
yum -y install centos-release-openstack-train
sed -i -e "s/enabled=0/enabled=1/g" /etc/yum.repos.d/CentOS-OpenStack-train.repo
```
- Cài đặt NTP để đồng bộ thời gian trên các node
```
yum -y install chrony
sed -i 's/server 0.centos.pool.ntp.org iburst/ \
server 1.vn.pool.ntp.org iburst \
server 0.asia.pool.ntp.org iburst \
server 3.asia.pool.ntp.org iburst/g' /etc/chrony.conf
sed -i 's/server 1.centos.pool.ntp.org iburst/#/g' /etc/chrony.conf
sed -i 's/server 2.centos.pool.ntp.org iburst/#/g' /etc/chrony.conf
sed -i 's/server 3.centos.pool.ntp.org iburst/#/g' /etc/chrony.conf
sed -i 's/#allow 192.168.0.0\/16/allow 172.16.0.0\/16/g' /etc/chrony.conf
```
- Đồng bộ thời gian
```
systemctl enable chronyd.service
systemctl start chronyd.service
chronyc sources
```




