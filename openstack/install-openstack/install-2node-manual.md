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
``` 

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

<a name="thietlap"></a>
## 4. Cài đặt node controller

***4.1 Cài đặt RAbbitMQ, Memcached, MariaDB***

```
yum install -y memcached  mariadb mariadb-server python2-PyMySQL rabbitmq-server
```

- Chỉnh sửa config memcached 
```
cp /etc/sysconfig/memcached /etc/sysconfig/memcached.orig
sed -i "s/-l 127.0.0.1,::1/ 0.0.0.0,::/g" /etc/sysconfig/memcached
```

- Chỉnh sửa cấu hình mariadb 
```
cp /etc/my.cnf.d/server.cnf /etc/my.cnf.d/server.cnf.orig
cat << EOF >> /etc/my.cnf.d/openstack.cnf 
[mysqld]
bind-address = 0.0.0.0
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
EOF
```
Lưu ý : Đặt pasword cho user mysql 
###### mysql_secure_installation ####

- Cấu hình rabbitMQ

```
systemctl enable rabbitmq-server.service
systemctl start rabbitmq-server.service
rabbitmq-plugins enable rabbitmq_management
systemctl restart rabbitmq-server
curl -O http://localhost:15672/cli/rabbitmqadmin
chmod a+x rabbitmqadmin
mv rabbitmqadmin /usr/sbin/
rabbitmqadmin list users
rabbitmqctl add_user openstack Welcome123
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
rabbitmqctl set_user_tags openstack administrator
```
- Khởi động các service lên 
```
systemctl enable mariadb.service
systemctl restart mariadb.service
systemctl enable memcached.service
systemctl restart memcached.service
```

***4.2 Cài đặt Keystone***

- Tạo Database
```
mysql -u root -proot
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'keystone';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'keystone';
flush privileges;
```

- Cài đặt các gói liên quan
```
yum install openstack-keystone httpd mod_wsgi -y
```

- Cấu hình keystone 
```
cp /etc/keystone/keystone.conf /etc/keystone/keystone.conf.org
Trong phần [database] , [token] chỉnh sửa
[database]
# ...
connection = mysql+pymysql://keystone:keystone@controller/keystone
[token]
# ...
provider = fernet
```
- Phân quyền file cấu hình 
```
chown root:keystone /etc/keystone/keystone.conf
```
- Sync DB
```
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

- Cài repo fernet key
```
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

- Bootstrap keystone 
```
keystone-manage bootstrap --bootstrap-password admin \
  --bootstrap-admin-url http://controller:5000/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```

- Cấu hình Apache http server
```
vi /etc/httpd/conf/httpd.conf
ServerName controller
```

- Tạo link tới file config wsgi 
```
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
```
- Bật dịch vụ
```
systemctl start httpd
systemctl enable httpd
```

- Thiết lập biến môi trường 
```
export OS_USERNAME=admin
export OS_PASSWORD=admin
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
```
##### Lưu ý giá trị OS_password được thiết lập ở trên keystone-manage bootstrap #####
### Verify lại keystone ###
- Tạo domain
```
openstack domain create --description "An Example Domain" example
```
- Tạo project 
```
openstack project create service --domain default --description "Service Project
openstack project create demo --domain default --description "Demo Project" 
```
- tạo user 
```
openstack user create demo --domain default --password demo
```
- tạo role 
```
openstack role create user
```
- Gán role vào project và user
```
openstack role add --project demo --user demo user
openstack role add --project service --user admin admin
```
- Gỡ biến môi trường
```
unset OS_AUTH_URL OS_PASSWORD
```
- Lấy token cho user admin
```
openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue
```

- Thành công ta sẽ tạo scripts chứa biến môi trường 
```
cat << EOF > /root/admin-openrc
export OS_USERNAME=admin
export OS_PASSWORD=admin
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://172.16.4.200:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```
- Sử dụng script
```
. admin-openrc
openstack token issue
```

***4.3 Cài đặt Glance***
- Tạo databases
```
CREATE DATABASE glance;
```












