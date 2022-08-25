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
- Nhập biến môi trường xác thực với keystone
```
export OS_USERNAME=admin
export OS_PASSWORD=admin
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
```

**** Lưu ý **** 
---
![image](https://user-images.githubusercontent.com/50499526/186299650-9619baf8-491f-4e48-b402-4aa31bf3c3d4.png)

Nếu gặp lỗi như hình thì chuyển sang 1 máy linux khác và thực hiện SSH vào máy controller để cài đặt tiếp 
Lỗi liên quan tới giao diện windows
---
- tạo 1 domain 
```
openstack domain create --description "An Example Domain" example
```
- Mặc định đã có 1 domain là "default" sau khi ta tạo boostrap 
- Tạo 1 project 
```
openstack project create --domain default  --description "Service Project" service
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
---
- Cài đặt Glance
- Tạo Database
```
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'GLANCE_DBPASS';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'GLANCE_DBPASS';
```
- Tạo user glance 
```
openstack user create glance --domain default --password glance
```
- Thêm role cho user glance 
```
openstack role add --project service --user glance admin
```
- tạo service glance 
```
openstack service create --name glance --description "OpenStack Image" image
```
- tạo endpoint
```
openstack endpoint create --region RegionOne image public http://controller:9292
openstack endpoint create --region RegionOne image internal http://controller:9292
openstack endpoint create --region RegionOne image admin http://controller:9292
```
- cài đặt glance và các gói phụ thuộc 
```
apt install glance -y
```
- chỉnh sửa các section trong file /etc/glance/glance-api.conf
```
[database]
# ...
connection = mysql+pymysql://glance:glance@controller/glance
[keystone_authtoken]
# ...
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = glance
[paste_deploy]
# ...
flavor = keystone
[glance_store]
# ...
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
```
- đồng bộ database
```
 su -s /bin/sh -c "glance-manage db_sync" glance
```
- Khởi động lại service 
```
service glance-api restart
```
- Tài về image và đẩy lên glance
```
wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img
glance image-create --name "cirros" --file cirros-0.4.0-x86_64-disk.img --disk-format qcow2 --container-format bare --visibility=public
glance image-list
```
---
- Cài đặt placement
- Tạo DB
```
CREATE DATABASE placement;
GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY 'PLACEMENT_DBPASS';
GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY 'PLACEMENT_DBPASS';
```
- Tạo user cho service placement và thêm quyền 
```
openstack user create placement --domain default --password placement
openstack role add --project service --user placement admin
```
- tạo service placement và endpoint
```
openstack service create --name placement --description "Placement API" placement
openstack endpoint create --region RegionOne placement public http://controller:8778
openstack endpoint create --region RegionOne placement internal http://controller:8778
openstack endpoint create --region RegionOne placement admin http://controller:8778
```
- cài các gói liên quan
```
apt install placement-api -y
```
- chỉnh sửa file cấu hình /etc/placement/placement.conf
```
[placement_database]
# ...
connection = mysql+pymysql://placement:placement@controller/placement
[api]
# ...
auth_strategy = keystone

[keystone_authtoken]
# ...
auth_url = http://controller:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = placement
password = placement
```
- Đồng bộ DB và chạy service
```
su -s /bin/sh -c "placement-manage db sync" placement
service apache2 restart
```
- Cài đặt nova
- Tạo database
```
CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'nova';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'nova';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'nova';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'nova';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'nova';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'nova';
```
- tạo user nova và phân quyền 
```
openstack user create nova --domain default --password nova
openstack role add --project service --user nova admin
```
- Tạo service và endpoint
```
openstack service create --name nova --description "OpenStack Compute" compute
openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
```
- Cài các gói liên quan
```
apt install nova-api nova-conductor nova-novncproxy nova-scheduler -y
```
- Chỉnh sửa cấu hình trong file /etc/nova/nova.conf
```
[api_database]
# ...
connection = mysql+pymysql://nova:nova@controller/nova_api
[database]
# ...
connection = mysql+pymysql://nova:nova@controller/nova
[DEFAULT]
# ...
transport_url = rabbit://openstack:RABBIT_PASS@controller:5672/
my_ip = 172.16.4.200
[api]
# ...
auth_strategy = keystone
[keystone_authtoken]
# ...
www_authenticate_uri = http://controller:5000/
auth_url = http://controller:5000/
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = nova
[vnc]
enabled = true
# ...
server_listen = $my_ip
server_proxyclient_address = $my_ip
[glance]
# ...
api_servers = http://controller:9292
[oslo_concurrency]
# ...
lock_path = /var/lib/nova/tmp
[placement]
# ...
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = placement
```
- đồng bộ DB
```
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
su -s /bin/sh -c "nova-manage db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
```
- Khởi chạy dịch vu
```
service nova-api restart
service nova-scheduler restart
service nova-conductor restart
service nova-novncproxy restart
```
- cài đặt neutron
- tạo database
```
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'neutron';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'neutron';
```
- tạo user và thêm quyền
```
openstack user create neutron --domain default --password neutron
openstack role add --project service --user neutron admin
```
- tạo service và endpoint
```
openstack service create --name neutron --description "OpenStack Networking" network
openstack endpoint create --region RegionOne network public http://controller:9696
openstack endpoint create --region RegionOne network internal http://controller:9696
openstack endpoint create --region RegionOne network admin http://controller:9696
```
***Lưu ý  ***
- Có 2 tùy chọn loại mạng cho neutron là provider và self-service
- Ở đây chọn sử dụng provider network
***************
- cài đặt các gói  liên quan
```
apt install neutron-server neutron-plugin-ml2  neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent -y
```
- chỉnh sửa các section trong file cấu hình /etc/neutron/neutron.conf
```
[database]
# ...
connection = mysql+pymysql://neutron:neutron@controller/neutron
[DEFAULT]
# ...
core_plugin = ml2
service_plugins =
transport_url = rabbit://openstack:openstack@controller
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
[keystone_authtoken]
# ...
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = neutron
[nova]
# ...
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = nova
[oslo_concurrency]
# ...
lock_path = /var/lib/neutron/tmp
```
- Cấu hình module layer2 trong file /etc/neutron/plugins/ml2/ml2_conf.ini
```
[ml2]
type_drivers = flat,vlan
mechanism_drivers = linuxbridge
tenant_network_types =
extension_drivers = port_security
[ml2_type_flat]
flat_networks = provider
[securitygroup]
enable_ipset = true
```
- Cấu hình file /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```
[linux_bridge]
physical_interface_mappings = provider:PROVIDER_INTERFACE_NAME
[vxlan]
enable_vxlan = false
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
- Cấu hình file /etc/neutron/dhcp_agent.ini
```
[DEFAULT]
# ...
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
```
- Chỉnh cấu hình nova kết nối với neutron trong file /etc/nova/nova.conf
```
[neutron]
# ...
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = neutron
service_metadata_proxy = true
metadata_proxy_shared_secret = METADATA_SECRET
```
- Đồng bộ DB và khởi chạy service
```
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
service nova-api restart
service neutron-server restart
service neutron-linuxbridge-agent restart
```
- cài đặt horizon
```
apt install openstack-dashboard -y 
```
- Chỉnh sửa cấu hình trong file /etc/openstack-dashboard/local_settings.py
```
OPENSTACK_HOST = "controller"
ALLOWED_HOSTS = ['*']
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'controller:11211',
    }
}
OPENSTACK_KEYSTONE_URL = "http://%s/identity/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 3,
}
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
OPENSTACK_NEUTRON_NETWORK = {
    ...
    'enable_router': False,
    'enable_quotas': False,
    'enable_ipv6': False,
    'enable_distributed_router': False,
    'enable_ha_router': False,
    'enable_fip_topology_check': False,
}
TIME_ZONE = "Asia/Ho_Chi_Minh"
```
- Thêm cấu hình trong /etc/apache2/conf-available/openstack-dashboard.conf
```
WSGIApplicationGroup %{GLOBAL}
```
- Reload lại httpd
```
systemctl reload apache2.service
```

