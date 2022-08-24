# Hướng dẫn cài đặt OpenStack Yoga trên Ubuntu 20.04 LTS
## Mục lục

[1. Chuẩn bị](#prepare)

[2. Setup môi trường cài đặt](#set-up)

- [2.2. Cài đặt trên controller](#controller)
- [2.3. Cài đặt trên  compute node](#compute)

<a name="mohinh"></a>
## 1. Thiết lập ban đầu 
***Làm trên tất cả các node***
- Set hostname 
- Cài đặt NTP 

``` apt install chrony -y ```
- Cấu hình  NTP đồng bộ thời gian

``` 
timedatectl set-timezone Asia/Ho_Chi_Minh
vi /etc/chrony/chrony.conf
server 1.vn.pool.ntp.org
server 2.asia.pool.ntp.org
server 0.asia.pool.ntp.org 
allow 172.16.0.0/16
```
- Restart service 
```
service chrony restart
systemctl enable chrony
```
- Tắt ufw
```
systemctl stop ufw
systemctl disable ufw 
```
- - Khai báo file host các node 
``` 
echo "172.16.4.200 controller" >> /etc/hosts
echo "172.16.4.201 compute01" >> /etc/hosts
echo "172.16.4.202 compute02" >> /etc/hosts
```
- cài đặt client để thao tác commandline
```
apt install python3-openstackclient -y
```
## 2. Cấu hình node controller 
- Add repository openstack yoga
```
add-apt-repository cloud-archive:yoga
```
- Cài đặt SQL 
```
apt install mariadb-server python3-pymysql -y
```
- Tạo và chỉnh sửa file /etc/mysql/mariadb.conf.d/99-openstack.cnf 
- Thêm section [mysqld]
```
[mysqld]
bind-address = 0.0.0.0
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```
- Khởi động lại dịch vụ
```
service mysql restart
```
- Đặt mật khẩu cho database
```
mysql_secure_installation
```
- Cài đặt message queue 
```
 apt install rabbitmq-server -y 
```
- thêm user openstack vào rabbitmq
```
rabbitmqctl add_user openstack RABBIT_PASS
```
- cấp quyền cho user openstack
```
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```
- Cài đặt memcached
```
apt install memcached python3-memcache
```
- chỉnh sửa file /etc/memcached.conf
```
-l 0.0.0.0
```
- Khởi động service 
```
service memcached restart
```
- Cài dặt ETCD ( dịch vụ hỗ trợ lưu trữ , phân tán khóa , lưu cấu hình và theo dõi hoạt động các dịch vụ )
```
apt install etcd -y
```
- Cấu hình file etc/default/etcd
```
ETCD_NAME="controller"
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER="controller=http://<IP_controller>:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://<IP_controller>:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://<IP_controller>:2379"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://<IP_controller>:2379"
```
- khởi động service 
```
systemctl enable etcd
systemctl restart etcd
```
- Cài đặt Keystone 
- Tạo database
```
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'KEYSTONE_DBPASS';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'KEYSTONE_DBPASS';
```
- cài đặt packages
```
apt install keystone
```
***Lưu ý***: Trong package này đã cài sẵn apache2 và mod_wsgi chạy trên port 5000 phục vụ cho identity service 
- Chỉnh sửa các section trong file /etc/keystone/keystone.conf
```
[database]
# ...
connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
[token]
# ...
provider = fernet
```
- Đồng bộ database
```
su -s /bin/sh -c "keystone-manage db_sync" keystone
```
- tạo fernet-key 
```
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```
- tạo boostrap 
```
keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
  --bootstrap-admin-url http://controller:5000/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```
- chỉnh sửa file /etc/apache2/apache2.conf
```
ServerName controller
```
- khởi động lại service 
```
service apache2 restart
```
