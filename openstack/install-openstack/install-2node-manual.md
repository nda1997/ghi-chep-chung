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
- set hostname cho từng node 
```
hostnamectl set-hostname controller 
```
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
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

```
Kiểm tra lại 
``` sysctl -p ```

#### Nếu báo lỗi thì là do thiếu module dưới kernel . thực hiện thêm 2 module sau 
```
modprobe br_netfilter
modprobe ip_conntrack
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
cp /etc/my.cnf.d/mariadb-server.cnf /etc/my.cnf.d/mariadb-server.cnf.orig
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
- Ở đây tôi đặt luôn là root
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
rabbitmqctl add_user openstack openstack
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
exit;
```

- Cài đặt các gói liên quan
```
yum install openstack-keystone openstack-utils python-openstackclient httpd mod_wsgi -y
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

- Cài fernet key
```
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

- Bootstrap keystone 
```
keystone-manage bootstrap \
    --bootstrap-password admin \
    --bootstrap-username admin \
    --bootstrap-project-name admin \
    --bootstrap-role-name admin \
    --bootstrap-service-name keystone \
    --bootstrap-region-id RegionOne \
    --bootstrap-admin-url http://controller1:5000/v3/ \
    --bootstrap-public-url http://controller1:5000/v3/ \
    --bootstrap-internal-url http://controller1:5000/v3/
```

- Cấu hình /etc/httpd/conf/httpd.conf
```
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
openstack project create service --domain default --description "Service Project"
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
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'glance';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'glance';
```
- Tạo user
```
openstack user create  glance --domain default --password glance
openstack role add --project service --user glance admin
```
- Tạo endpoint
```
openstack service create --name glance --description "OpenStack Image" image
openstack endpoint create --region RegionOne image public http://172.16.4.200:9292
openstack endpoint create --region RegionOne image internal http://172.16.4.200:9292
openstack endpoint create --region RegionOne image admin http://172.16.4.200:9292
```
- Cài đặt 
```
yum install openstack-glance MySQL-python python-devel
```
- Cấu hình file glance-api
```
cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.orig
[database]
# ...
connection = mysql+pymysql://glance:glance@controller/glance
[keystone_authtoken]
# ...
www_authenticate_uri  = http://controller:5000
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
- Đồng bộ databases
```
su -s /bin/sh -c "glance-manage db_sync" glance
```
- Khởi động service
```
systemctl enable openstack-glance-api.service
systemctl start openstack-glance-api.service
```
- Test glance 
```
wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img
openstack image create "cirros" --file cirros-0.4.0-x86_64-disk.img --disk-format qcow2 --container-format bare --public
```
***4.4 Cài đặt placement***
- Tạo DB
```
CREATE DATABASE placement;
GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY 'placement';
GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY 'placement';
flush privileges;
exit;
```
- Tạo user và phần quyền 
```
openstack user create placement --domain default --password placement
openstack role add --project service --user placement admin
```
- Tạo service và endpoint
```
openstack service create --name placement --description "Placement API" placement
openstack endpoint create --region RegionOne placement public http://controller:8778
openstack endpoint create --region RegionOne placement internal http://controller:8778
openstack endpoint create --region RegionOne placement admin http://controller:8778
```
- Cài đặt 
```
yum install openstack-placement-api -y
```
- Cấu hình file /etc/placement/placement.conf
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
- đồng bộ DB
```
su -s /bin/sh -c "placement-manage db sync" placement
```
- Restart HTTPD
```
systemctl restart httpd
```
***4.5 Cài đặt Nova***
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
flush privileges;
exit;
```
- Tạo user và phần quyền
```
openstack user create nova --domain default --password nova
openstack role add --project service --user nova admin
```
- Tạo endpoint
```
openstack service create --name nova --description "OpenStack Compute" compute
openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
```
- Cài đặt các gói 
```
yum install openstack-nova-api openstack-nova-conductor openstack-nova-novncproxy openstack-nova-scheduler
```
- Cấu hình nova chỉnh sửa các section dưới 
```
vi /etc/nova/nova.conf
[DEFAULT]
# ...
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:openstack@controller:5672/
my_ip = 172.16.4.200
use_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[api_database]
# ...
connection = mysql+pymysql://nova:nova@controller/nova_api
[database]
# ...
connection = mysql+pymysql://nova:nova@controller/nova
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
[wsgi]
api_paste_config = /etc/nova/api-paste.ini
```
- Đồng bộ DB
```
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
su -s /bin/sh -c "nova-manage db sync" nova
Xác nhận lại nova cell0 ,cell1 đã chạy thành công chưa
su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
```
- Khời động service
```
systemctl enable openstack-nova-api.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
systemctl start openstack-nova-api.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
```
***4.6 Cài đặt Cinder***
- Tạo DB
```
CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY 'cinder';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' IDENTIFIED BY 'cinder';
```
- Tạo user và phân quyền 
```
openstack user create cinder --domain default --password cinder
openstack role add --project service --user cinder admin
```
- Tạo service và endpoint
```
openstack service create --name cinderv2 --description "OpenStack Block Storage" volumev2
openstack service create --name cinderv3 --description "OpenStack Block Storage" volumev3
openstack endpoint create --region RegionOne volumev2 public http://controller:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne volumev2 internal http://controller:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne volumev2 admin http://controller:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne volumev3 public http://controller:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne volumev3 internal http://controller:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne volumev3 admin http://controller:8776/v3/%\(project_id\)s
```
- Cài đặt 
```
yum install openstack-cinder lvm2 device-mapper-persistent-data  targetcli python-keystone
```
- Tạo phân vùng lưu trữ lvm cho cinder 
****Ở đây ta attach thêm ổ cứng vào controller****
```
pvcreate /dev/sdb
vgcreate cinder-volumes /dev/sdb
```
- Cấu hình /etc/lvm/lvm.conf để nhận volume-group vừa tạo 
```
devices {
...
filter = [ "a/.*/" ]
```
- Cấu hình cinder 
```
/etc/cinder/cinder.conf
[DEFAULT]
# ...
transport_url = rabbit://openstack:openstack@controller
auth_strategy = keystone
my_ip = 172.16.4.200
enabled_backends = lvm
glance_api_servers = http://controller:9292
[database]
# ...
connection = mysql+pymysql://cinder:cinder@controller/cinder
[keystone_authtoken]
# ...
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = cinder
[oslo_concurrency]
# ...
lock_path = /var/lib/cinder/tmp
[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
target_protocol = iscsi
target_helper = lioadm
```
- Đồng bộ DB
```
su -s /bin/sh -c "cinder-manage db sync" cinder
```
- Khởi động service
```
systemctl enable openstack-cinder-volume.service target.service lvm2-lvmetad.service openstack-cinder-api.service openstack-cinder-scheduler.service
systemctl start openstack-cinder-volume.service target.service lvm2-lvmetad.service openstack-cinder-api.service openstack-cinder-scheduler.service
```
- thêm cấu hình /etc/nova/nova.conf
```
[cinder]
os_region_name = RegionOne
```
***4.7 Cài đặt neutron***
- Tạo DB
```
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'neutron';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'neutron';
```
- Tạo user và phân quyền 
```
openstack user create neutron --domain default --password neutron
openstack role add --project service --user neutron admin
```
- Tạo service và endpoint
```
openstack service create --name neutron --description "OpenStack Networking" network
openstack endpoint create --region RegionOne network public http://controller:9696
openstack endpoint create --region RegionOne network internal http://controller:9696
openstack endpoint create --region RegionOne network admin http://controller:9696
```
#### Có 2 mô hình mạng là self-service và provider ####
##### Cài đặt theo kiến trúc provider #####
- Cài đặt các gói neutron
```
yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables 
```
- Chỉnh sửa file /etc/neutron/neutron.conf
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
- Cấu hình file /etc/neutron/plugins/ml2/ml2_conf.ini
```
[ml2]
# ...
type_drivers = flat,vlan
tenant_network_types =
mechanism_drivers = linuxbridge
extension_drivers = port_security
[ml2_type_flat]
# ...
flat_networks = provider
[securitygroup]
# ...
enable_ipset = true
```
- Cấu hình file /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```
[linux_bridge]
physical_interface_mappings = provider:PROVIDER_INTERFACE_NAME ( Ở đây có 1 dải mạng nên để là eth0 )
[vxlan]
enable_vxlan = false
[securitygroup]
# ...
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
- thêm vào file /etc/nova/nova.conf
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
password = NEUTRON_PASS
service_metadata_proxy = true
metadata_proxy_shared_secret = admin
```

##### Tùy chọn 2 : sử dụng mạng self-service #####
- Chỉnh sửa file /etc/neutron/neutron.conf
```
[database]
# ...
connection = mysql+pymysql://neutron:neutron@controller/neutron
[DEFAULT]
# ...
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
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

- Chỉnh sửa file /etc/neutron/plugins/ml2/ml2_conf.ini
```
[ml2]
# ...
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security
[ml2_type_flat]
# ...
flat_networks = provider
[ml2_type_vxlan]
# ...
vni_ranges = 1:1000
[securitygroup]
# ...
enable_ipset = true
```
*****Lưu ý sau khi configure ml2 plug-in thì xóa bỏ dòng type_driver để tránh việc không nhất quán database*****
-----------------------------------------------------------
- chỉnh sửa file /etc/neutron/dhcp_agent.ini
```
[DEFAULT]
# ...
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
```
- chỉnh sửa file /etc/neutron/metadata_agent.ini
```
[DEFAULT]
# ...
nova_metadata_host = controller
metadata_proxy_shared_secret = admin
```
- Chỉnh sửa file /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```
[linux_bridge]
physical_interface_mappings = provider:PROVIDER_INTERFACE_NAME
[vxlan]
enable_vxlan = true
local_ip = OVERLAY_INTERFACE_IP_ADDRESS
l2_population = true
[securitygroup]
# ...
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
- chỉnh sửa file /etc/neutron/l3_agent.ini
```
[DEFAULT]
# ...
interface_driver = linuxbridge
```
- tạo symbolic link
```
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```
- đồng bộ db
```
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```
- khởi chạy dịch vụ
```
systemctl restart openstack-nova-api.service
systemctl enable neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service
systemctl start  neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service
```

***4.8 Cài đặt horizon***
- cài đặt 
```
yum install openstack-dashboard -y
```
- sửa file /etc/openstack-dashboard/local_settings
```
OPENSTACK_HOST = "controller"
ALLOWED_HOSTS = ['*',]
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'controller:11211',
    }
}
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 3,
}
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
TIME_ZONE = "TIME_ZONE"
WEBROOT = '/dashboard/'
*****Nếu chọn provider network thêm đoạn sau để disable support layer3-network*****
OPENSTACK_NEUTRON_NETWORK = {
    'enable_router': False,
    'enable_quotas': False,
    'enable_distributed_router': False,
    'enable_ha_router': False,
    'enable_lb': False,
    'enable_firewall': False,
    'enable_vpn': False,
    'enable_fip_topology_check': False,
}
```
- Thêm dòng sau vào file /etc/httpd/conf.d/openstack-dashboard.conf
```
WSGIApplicationGroup %{GLOBAL}
```
- Khởi động lại service
```
systemctl restart httpd.service memcached.service
```

<a name="compute"></a>
## 5. Cài đặt node compute

***5.1 cài đặt nova-compute***
- install package
```
yum install openstack-nova-compute -y
```
- chỉnh sửa file  /etc/nova/nova.conf
```
[DEFAULT]
# ...
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:openstack@controller
my_ip = 172.16.4.200
use_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver
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
# ...
enabled = true
server_listen = 0.0.0.0
server_proxyclient_address = $my_ip
novncproxy_base_url = http://172.16.4.200:6080/vnc_auto.html
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
- Khởi chạy service 
```
systemctl enable libvirtd.service openstack-nova-compute.service
systemctl start libvirtd.service openstack-nova-compute.service
```

***5.2 Cài đặt neutron ***
- install package 
```
yum install openstack-neutron-ml2 openstack-neutron  openstack-neutron-linuxbridge ebtables -y 
```
- Sửa file /etc/neutron/neutron.conf
```
[DEFAULT]
# ...
transport_url = rabbit://openstack:openstack@controller
auth_strategy = keystone
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
[oslo_concurrency]
# ...
lock_path = /var/lib/neutron/tmp
```

- Khai báo neutron trong nova /etc/nova/nova.conf
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
service_metadata_proxy = True
metadata_proxy_shared_secret = admin
```

***Lựa chọn 1 : provider network ***
- Sửa file /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```
[linux_bridge]
physical_interface_mappings = provider:PROVIDER_INTERFACE_NAME

[vxlan]
enable_vxlan = false
[securitygroup]
# ...
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
***Lựa chọn 2 : self-service network ***
- Sửa file /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```
[linux_bridge]
physical_interface_mappings = provider:PROVIDER_INTERFACE_NAME
[vxlan]
enable_vxlan = true
local_ip = OVERLAY_INTERFACE_IP_ADDRESS 
l2_population = true
[securitygroup]
# ...
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
---------------------------------------------------------------
- Cấu hình /etc/neutron/dhcp_agent.ini
```
[DEFAULT]
# ...
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
```
- Cấu hình /etc/neutron/metadata_agent.ini
```
[DEFAULT]
nova_metadata_host = 172.16.4.200
metadata_proxy_shared_secret = admin
memcache_servers = 172.16.4.200:11211
```
- Khởi chạy service 
```
systemctl restart openstack-nova-compute.service
systemctl enable neutron-linuxbridge-agent.service neutron-dhcp-agent neutron-metadata-agent
systemctl start neutron-linuxbridge-agent.service neutron-dhcp-agent neutron-metadata-agent
```
