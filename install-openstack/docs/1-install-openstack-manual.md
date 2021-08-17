# Cài đặt OpenStack Manual trên CentOS 7

## Mục lục

[Phần 1. Chuẩn bị](https://github.com/quanganh1996111/openstack/blob/main/install-openstack/docs/1-install-openstack-manual.md#ph%E1%BA%A7n-1-chu%E1%BA%A9n-b%E1%BB%8B)

- [1. Mô hình triển khai](https://github.com/quanganh1996111/openstack/blob/main/install-openstack/docs/1-install-openstack-manual.md#1-m%C3%B4-h%C3%ACnh-tri%E1%BB%83n-khai)

- [2. Danh sách IP](https://github.com/quanganh1996111/openstack/blob/main/install-openstack/docs/1-install-openstack-manual.md#2-danh-s%C3%A1ch-ip)

- [3. Bật chế độ ảo hóa trên KVM](https://github.com/quanganh1996111/openstack/blob/main/install-openstack/docs/1-install-openstack-manual.md#3-b%E1%BA%ADt-ch%E1%BA%BF-%C4%91%E1%BB%99-%E1%BA%A3o-h%C3%B3a-tr%C3%AAn-kvm)

[Phần 2. Cấu hình Node Controller](https://github.com/quanganh1996111/openstack/blob/main/install-openstack/docs/1-install-openstack-manual.md#ph%E1%BA%A7n-2-c%E1%BA%A5u-h%C3%ACnh-node-controller)

- [1. Cấu hình cơ bản]()

## Phần 1. Chuẩn bị

### 1.1. Mô hình triển khai

Mô hình triển khai gồm 1 node Controller, 2 node Compute.

![](../images/1-install-manual-ops/install-manual-ops.png)

### 1.2. Danh sách IP

![](../images/1-install-manual-ops/IP-planning-manual.png)

### 1.3. Bật chế độ ảo hóa trên KVM

Ở đây triển khai trên môi trường ảo hóa KVM nên ta phải bật mode ảo hóa đối với máy ảo được tạo ra.

**Thực hiện trên Node KVM vật lý**

- Kiểm tra xem ảo hóa đã được enable hay chưa:

```
- Đối với Server sử dụng chip Intel:
cat /sys/module/kvm_intel/parameters/nested

- Đối với Server sử dụng chip AMD:
cat /sys/module/kvm_amd/parameters/nested
```

- Kết quả thường sẽ trả về là `N`:

```
[root@localhost ~]# cat /sys/module/kvm_intel/parameters/nested
N
```

**Lưu ý:** Tiến hành `shutdown` tất cả các VM trên Server KVM.

- Để enable ảo hóa trên KVM, tạo một file có đường dẫn `/etc/modprobe.d/kvm-nested.conf` và thêm cấu hình như sau:

```
[root@localhost ~]# vi /etc/modprobe.d/kvm-nested.conf
options kvm-intel nested=1
options kvm-intel enable_shadow_vmcs=1
options kvm-intel enable_apicv=1
options kvm-intel ept=1
```

- Tiến hành Remove `kvm_intel` module sau đó thêm lại:

```
[root@localhost ~]# modprobe -r kvm_intel //Xóa kvm_intel module
[root@localhost ~]# modprobe -a kvm_intel //Thêm kvm_intel module
```

- Khởi động các VM lên và tiến hành kiểm tra:

```
[root@localhost ~]# cat /proc/cpuinfo | egrep -c "vmx|svm"
2
[root@localhost ~]# lscpu
```

Kết quả trả về `Virtualization type:   full` là thành công.

![](../images/1-install-manual-ops/test-module-virtual-kvm.png)

Có thể tham khảo thêm việc enable ảo hóa trên Server sử dụng chip AMD [tại đây](https://www.linuxtechi.com/enable-nested-virtualization-kvm-centos-7-rhel-7/)

## Phần 2. Cấu hình Node Controller

### 1. Cấu hình cơ bản

#### Cấu hình IP

```
hostnamectl set-hostname controller
sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl disable NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl enable network
sudo systemctl start network
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

#### Cấu hình các mode sysctl

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

Kiểm tra lại cấu hình:

```
sysctl -p
```

#### Khai báo repo mariadb và update

```
echo '[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1' >> /etc/yum.repos.d/MariaDB.repo
yum -y update
```

#### Khai báo file hosts các node

```
echo "172.16.2.56 controller" >> /etc/hosts
echo "172.16.2.59 compute01" >> /etc/hosts
echo "172.16.2.62 compute02" >> /etc/hosts
```

#### Tạo SSH key và coppy sang các node compute

```
ssh-keygen -t rsa -f /root/.ssh/id_rsa -q -P ""
ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@controller
ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@compute01
ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@compute02
scp /root/.ssh/id_rsa root@compute01:/root/.ssh/
scp /root/.ssh/id_rsa root@compute02:/root/.ssh/
```

Đứng từ `controller` tiến hành SSH đến 2 node `compute`

![](../images/1-install-manual-ops/ssh-compute.png)

#### Cài đặt các gói cần thiết

```
yum -y install centos-release-openstack-queens
yum -y install crudini wget vim
yum -y install python-openstackclient openstack-selinux python2-PyMySQL
```

### 2. Cài đặt và cấu hình NTP

```
yum -y install chrony
sed -i 's/server 0.centos.pool.ntp.org iburst/ \
server 1.vn.pool.ntp.org iburst \
server 0.asia.pool.ntp.org iburst \
server 3.asia.pool.ntp.org iburst/g' /etc/chrony.conf
sed -i 's/server 1.centos.pool.ntp.org iburst/#/g' /etc/chrony.conf
sed -i 's/server 2.centos.pool.ntp.org iburst/#/g' /etc/chrony.conf
sed -i 's/server 3.centos.pool.ntp.org iburst/#/g' /etc/chrony.conf
sed -i 's/#allow 192.168.0.0\/16/allow 10.10.10.0\/24/g' /etc/chrony.conf
```

```
systemctl enable chronyd.service
systemctl start chronyd.service
chronyc sources
```

### 3. Cài đặt và cấu hình memcache

- Cài đặt memcached:

```
yum install -y memcached
sed -i "s/-l 127.0.0.1,::1/-l 10.10.10.118/g" /etc/sysconfig/memcached
```

- Khởi động dịch vụ:

```
systemctl enable memcached.service
systemctl restart memcached.service
```

### 4. Cài đặt và cấu hình MariaDB

- Cài đặt:

```
yum install -y mariadb mariadb-server python2-PyMySQL
```

- Copy lại file cấu hình gốc:

```
cp /etc/my.cnf.d/server.cnf /etc/my.cnf.d/server.cnf.orig
rm -rf /etc/my.cnf.d/server.cnf
```

- Config file cấu hình mới:

```
cat << EOF > /etc/my.cnf.d/openstack.cnf
[mysqld]
bind-address = 172.16.2.56
default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
EOF
```

- Khởi động lại Dịch vụ:

```
systemctl enable mariadb.service
systemctl restart mariadb.service
```

- Đặt lại password cho user mysql

Lưu ý: Password đủ độ mạnh và tránh ký tự đặc biệt ở cuối như #,@

### 5. Cài đặt và cấu hình RabbitMQ

- Cài đặt:

```
yum -y install rabbitmq-server
```

- Cấu hình rabbitmq:

```
systemctl enable rabbitmq-server.service
systemctl start rabbitmq-server.service
rabbitmq-plugins enable rabbitmq_management
systemctl restart rabbitmq-server
curl -O http://localhost:15672/cli/rabbitmqadmin
chmod a+x rabbitmqadmin
mv rabbitmqadmin /usr/sbin/
rabbitmqadmin list users
```

- Cấu hình trên node controller:

```
rabbitmqctl add_user openstack 013279227Anh
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
rabbitmqctl set_user_tags openstack administrator
```

### 6. Cài đặt keystone

- Tạo Database:

```
mysql -u root -p
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY '013279227Anh';
FLUSH PRIVILEGES;
exit
```

- Cài đặt package:

```
yum install openstack-keystone httpd mod_wsgi -y
```

- Cấu hình bind port:

```
cp /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
sed -i -e 's/VirtualHost \*/VirtualHost 172.16.2.56/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/Listen 5000/Listen 172.16.2.56:5000/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/Listen 35357/Listen 172.16.2.56:35357/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/^Listen.*/Listen 172.16.2.56:80/g' /etc/httpd/conf/httpd.conf
```

- Cấu hình keystone:

```
cp /etc/keystone/keystone.conf /etc/keystone/keystone.conf.org
rm -rf /etc/keystone/keystone.conf
```

```
cat << EOF >> /etc/keystone/keystone.conf
[DEFAULT]
[assignment]
[auth]
[cache]
[catalog]
[cors]
[credential]
[database]
connection = mysql+pymysql://keystone:013279227Anh@172.16.2.56/keystone
[domain_config]
[endpoint_filter]
[endpoint_policy]
[eventlet_server]
[federation]
[fernet_tokens]
[healthcheck]
[identity]
[identity_mapping]
[ldap]
[matchmaker_redis]
[memcache]
[oauth1]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
[policy]
[profiler]
[resource]
[revoke]
[role]
[saml]
[security_compliance]
[shadow_users]
[signing]
[token]
provider = fernet
[tokenless_auth]
[trust]
EOF
```

- Phân quyền file cấu hình:

```
chown root:keystone /etc/keystone/keystone.conf
```

- Đồng bộ Database:

```
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

- Setup fernet key:

```
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

- Bootstrap keystone:

```
keystone-manage bootstrap --bootstrap-password 013279227Anh \
 --bootstrap-admin-url http://172.16.2.56:5000/v3/ \
 --bootstrap-internal-url http://172.16.2.56:5000/v3/ \
 --bootstrap-public-url http://172.16.2.56:5000/v3/ \
```

- Khởi động lại Dịch vụ HTTPD

```
systemctl enable httpd.service
systemctl restart httpd.service
```

- Export biến môi trường:

```
export OS_USERNAME=admin
export OS_PASSWORD=013279227Anh
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://172.16.2.56:35357/v3
export OS_IDENTITY_API_VERSION=3
```

- Tạo domain:

```
openstack domain create --description "An Example Domain" example
openstack project create --domain default --description "Service Project" service
```

- Tạo project và user:

```
openstack project create --domain default  --description "Demo Project" demo
openstack user create --domain default --password 013279227Anh demo
```

- Tạo role và gắn role:

```
openstack role create user
openstack role add --project demo --user demo user
```

- Unset 2 biến môi trường:

```
unset OS_AUTH_URL OS_PASSWORD
```

- Tạo token:

```
openstack --os-auth-url http://172.16.4.125:35357/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue
```

- Tạo file xác thực:

```
cat << EOF >> admin-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=013279227Anh
export OS_AUTH_URL=http://172.16.2.56:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```

```
cat << EOF >> demo-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=013279227Anh
export OS_AUTH_URL=http://172.16.2.56:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```

- Kiểm tra cấu hình keystone:

```
 . admin-openrc
openstack token issue
```

![](../images/1-install-manual-ops/token-issue.png)

### 7. Cài đặt và cấu hình glance

- Tạo Database:

```
mysql -u root -p
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY '013279227Anh';
exit
```

- Tạo user:

```
openstack user create --domain default --password 013279227Anh glance
openstack role add --project service --user glance admin
openstack service create --name glance --description "OpenStack Image" image
```

- Tạo endpoint:

```
openstack endpoint create --region RegionOne image public http://172.16.2.56:9292
openstack endpoint create --region RegionOne image admin http://172.16.2.56:9292
openstack endpoint create --region RegionOne image internal http://172.16.2.56:9292
```

- Cài packages:

```
yum install -y openstack-glance
```

- Cấu hình glance api:

```
cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.org 
rm -rf /etc/glance/glance-api.conf
```

```
cat << EOF >> /etc/glance/glance-api.conf
[DEFAULT]
bind_host = 172.16.2.56
registry_host = 172.16.2.56
[cors]
[database]
connection = mysql+pymysql://glance:013279227Anh@172.16.2.56/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[image_format]
[keystone_authtoken]
auth_uri = http://172.16.2.56:5000
auth_url = http://172.16.2.56:5000
memcached_servers = 172.16.2.56:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = 013279227Anh
[matchmaker_redis]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
[store_type_location_strategy]
[task]
[taskflow_executor]
EOF
```

- Cấu hình glance registry:

```
cp /etc/glance/glance-registry.conf /etc/glance/glance-registry.conf.org
rm -rf /etc/glance/glance-registry.conf
```

```
cat << EOF >> /etc/glance/glance-registry.conf
[DEFAULT]
bind_host = 172.16.2.56
[database]
connection = mysql+pymysql://glance:013279227Anh@172.16.2.56/glance
[keystone_authtoken]
auth_uri = http://172.16.2.56:5000
auth_url = http://172.16.2.56:5000
memcached_servers = 172.16.2.56
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = 013279227Anh
[matchmaker_redis]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
EOF
```

- Phân quyền file cấu hình:

```
chown root:glance /etc/glance/glance-api.conf
chown root:glance /etc/glance/glance-registry.conf
```

- Đồng bộ Database:

```
su -s /bin/sh -c "glance-manage db_sync" glance
```

- Enable và start dịch vụ:

```
systemctl enable openstack-glance-api.service openstack-glance-registry.service
systemctl start openstack-glance-api.service openstack-glance-registry.service
```

- Download và upload image test cirros:

```
wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
openstack image create "cirros" \
  --file cirros-0.3.5-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
```

Sau khi tạo images, mặc định image sẽ được đưa vào thư mục `/var/lib/glance/images`

![](../images/1-install-manual-ops/cirros-glance.png)

### 8. Cài đặt và cấu hình nova

- Tạo Databases:

```
mysql -u root -p013279227Anh
CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY '013279227Anh';
exit
```

- Tạo user và endpoint:

```
openstack user create --domain default --password 013279227Anh nova
openstack role add --project service --user nova admin
openstack service create --name nova --description "OpenStack Compute" compute
```

```
openstack endpoint create --region RegionOne compute public http://172.16.2.56:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://172.16.2.56:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://172.16.2.56:8774/v2.1
```

```
openstack user create --domain default --password Welcome123 placement
openstack role add --project service --user placement admin
openstack service create --name placement --description "Placement API" placement
```

```
openstack endpoint create --region RegionOne placement public http://172.16.2.56:8778
openstack endpoint create --region RegionOne placement admin http://172.16.2.56:8778
openstack endpoint create --region RegionOne placement internal http://172.16.2.56:8778
```

- Tải packages:

```
yum install -y openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler openstack-nova-placement-api
```

- Cấu hình nova:

```
cp /etc/nova/nova.conf /etc/nova/nova.conf.org 
rm -rf /etc/nova/nova.conf
```

```
cat << EOF >> /etc/nova/nova.conf
[DEFAULT]
my_ip = 172.16.2.56
enabled_apis = osapi_compute,metadata
use_neutron = True
osapi_compute_listen=172.16.2.56
metadata_host=172.16.2.56
metadata_listen=172.16.2.56
metadata_listen_port=8775
firewall_driver = nova.virt.firewall.NoopFirewallDriver
transport_url = rabbit://openstack:013279227Anh@172.16.2.56:5672
[api]
auth_strategy = keystone
[api_database]
connection = mysql+pymysql://nova:013279227Anh@172.16.2.56/nova_api
[barbican]
[cache]
backend = oslo_cache.memcache_pool
enabled = true
memcache_servers = 172.16.2.56:11211
[cells]
[cinder]
[compute]
[conductor]
[console]
[consoleauth]
[cors]
[crypto]
[database]
connection = mysql+pymysql://nova:013279227Anh@172.16.2.56/nova
[devices]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://172.16.2.56:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_url = http://172.16.2.56:5000/v3
memcached_servers = 172.16.2.56:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 013279227Anh
[libvirt]
[matchmaker_redis]
[metrics]
[mks]
[neutron]
[notifications]
[osapi_v21]
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
rabbit_ha_queues = true
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues= true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[pci]
[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://172.16.2.56:5000/v3
username = placement
password = 013279227Anh
[quota]
[rdp]
[remote_debug]
[scheduler]
discover_hosts_in_cells_interval = 300
[serial_console]
[service_user]
[spice]
[upgrade_levels]
[vault]
[vendordata_dynamic_auth]
[vmware]
[vnc]
novncproxy_host=172.16.2.56
enabled = true
vncserver_listen = 172.16.2.56
vncserver_proxyclient_address = 172.16.2.56
novncproxy_base_url = http://172.16.2.56:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
EOF
```

- Thêm vào file 00-nova-placement-api.conf:

```
cat << 'EOF' >> /etc/httpd/conf.d/00-nova-placement-api.conf

<Directory /usr/bin>
   <IfVersion >= 2.4>
      Require all granted
   </IfVersion>
   <IfVersion < 2.4>
      Order allow,deny
      Allow from all
   </IfVersion>
</Directory>
EOF
```

- Cấu hình bind port cho nova-placement:

```
sed -i -e 's/VirtualHost \*/VirtualHost 172.16.2.56/g' /etc/httpd/conf.d/00-nova-placement-api.conf
sed -i -e 's/Listen 8778/Listen 172.16.2.56:8778/g' /etc/httpd/conf.d/00-nova-placement-api.conf
```

```
systemctl restart httpd
```

- Đồng bộ Databases:

```
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
su -s /bin/sh -c "nova-manage db sync" nova
```

Lưu ý: Bỏ qua cảnh báo `Warning`

- Enable và start service:

```
systemctl enable openstack-nova-api.service openstack-nova-scheduler.service openstack-nova-consoleauth.service openstack-nova-conductor.service openstack-nova-novncproxy.service
systemctl start openstack-nova-api.service openstack-nova-scheduler.service openstack-nova-consoleauth.service openstack-nova-conductor.service openstack-nova-novncproxy.service
```

- Kiểm tra lại Dịch vụ:

```
openstack compute service list
```

![](../images/1-install-manual-ops/nova-services.png)

### 9. Cài đặt và cấu hình neutron

- Tạo database neutron:

```
mysql -u root -p
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY '013279227Anh';
exit
```

- Tạo user, endpoint trên 1 node:

```
openstack user create --domain default --password 013279227Anh neutron
openstack role add --project service --user neutron admin
openstack service create --name neutron --description "OpenStack Networking" network
```

```
openstack endpoint create --region RegionOne network public http://172.16.2.56:9696
openstack endpoint create --region RegionOne network internal http://172.16.2.56:9696
openstack endpoint create --region RegionOne network admin http://172.16.2.56:9696
```

- Cài đặt packages:

```
yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables -y
```

- Cấu hình neutron:

Bài lab này sử dụng mô hình mạng provider (flat) sử dụng linuxbridge DHCP agent và metadata agent được chạy trên node compute

```
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org
rm -rf /etc/neutron/neutron.conf
```

```
cat << EOF >> /etc/neutron/neutron.conf
[DEFAULT]
bind_host = 172.16.2.56
core_plugin = ml2
service_plugins = router
transport_url = rabbit://openstack:013279227Anh@172.16.2.56:5672
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
allow_overlapping_ips = True
dhcp_agents_per_network = 2
[agent]
[cors]
[database]
connection = mysql+pymysql://neutron:013279227Anh@172.16.2.56/neutron
[keystone_authtoken]
auth_uri = http://172.16.2.56:5000
auth_url = http://172.16.2.56:35357
memcached_servers = 172.16.2.56:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = 013279227Anh
[matchmaker_redis]
[nova]
auth_url = http://172.16.2.56:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = 013279227Anh
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues = true
rabbit_ha_queues = true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[quotas]
[ssl]
EOF
```

- Cấu hình file ml2:

```
cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.org
rm -rf /etc/neutron/plugins/ml2/ml2_conf.ini
```

```
cat << EOF >> /etc/neutron/plugins/ml2/ml2_conf.ini
[DEFAULT]
[l2pop]
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security
[ml2_type_flat]
[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]
network_vlan_ranges = provider
[ml2_type_vxlan]
vni_ranges = 1:1000
[securitygroup]
enable_ipset = True
EOF
```

- Cấu hình file LB agent:

```
cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini.org 
rm -rf /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```

Lưu ý khi chạy đoạn ở dưới chú ý 2 tham số:

physical_interface_mappings = provider:ens256 (interface name provider)

local_ip = 10.10.41.71(ip dải datavm controller)

```
cat << EOF >> /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:ens256
[network_log]
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = true
local_ip = 10.10.41.71
l2_population = true
EOF
```

- Cấu hình trên file l3 agent:

```
cp /etc/neutron/l3_agent.ini /etc/neutron/l3_agent.ini.org
rm -rf /etc/neutron/l3_agent.ini
```

```
cat << EOF >> /etc/neutron/l3_agent.ini
[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
[agent]
[ovs]
EOF
```

- Chỉnh sửa file `/etc/nova/nova.conf`

```
vi /etc/nova/nova.conf
```

```
[neutron]
url = http://172.16.2.56:9696
auth_url = http://172.16.2.56:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = 013279227Anh
service_metadata_proxy = true
metadata_proxy_shared_secret = 013279227Anh
```

- Restart lại service nova-api:

```
systemctl restart openstack-nova-api.service
```

- Phân quyền file cấu hình

```
chown -R root:neutron /etc/neutron/
```

- Tạo liên kết

```
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

- Sync db (bỏ qua các cảnh báo Warning):

```
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

- Enable và start dịch vụ

```
systemctl restart openstack-nova-api.service openstack-nova-scheduler.service openstack-nova-consoleauth.service openstack-nova-conductor.service openstack-nova-novncproxy.service
```

```
systemctl enable neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service
systemctl start neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service
```

### 10. Cài đặt và cấu hình horizon

- Tải packages:

```
yum install openstack-dashboard -y
```

- Tạo file direct:

```
filehtml=/var/www/html/index.html
touch $filehtml
cat << EOF >> $filehtml
<html>
<head>
<META HTTP-EQUIV="Refresh" Content="0.5; URL=http://172.16.2.56/dashboard">
</head>
<body>
<center> <h1>Redirecting to OpenStack Dashboard</h1> </center>
</body>
</html>
EOF
```

- Backup cấu hình:

```
cp /etc/openstack-dashboard/local_settings /etc/openstack-dashboard/local_settings.org
```

- Thay đổi cấu hình trong file `/etc/openstack-dashboard/local_settings`

```
ALLOWED_HOSTS = ['*',]
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'Default'
```

- Lưu ý thêm `SESSION_ENGINE` vào trên dòng `CACHE` như bên dưới:

```
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
         'BACKEND':'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': ['172.16.2.56:11211',],
    }
}
OPENSTACK_HOST = "172.16.2.56"
OPENSTACK_KEYSTONE_URL = "http://172.16.2.56:5000/v3"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
```

- Lưu ý: Nếu chỉ sử dụng provider, chỉnh sửa các thông số sau

```
OPENSTACK_NEUTRON_NETWORK = {
    'enable_router': False,
    'enable_quotas': False,
    'enable_ipv6': False,
    'enable_distributed_router': False,
    'enable_ha_router': False,
    'enable_fip_topology_check': False,
}
```

```
TIME_ZONE = "Asia/Ho_Chi_Minh"
```

- Phân quyền thư mục `/etc/openstack-dashboard/local_settings`

```
chown -R root:apache /etc/openstack-dashboard/local_settings
```

- Thêm vào file `/etc/httpd/conf.d/openstack-dashboard.conf`

```
echo "WSGIApplicationGroup %{GLOBAL}" >> /etc/httpd/conf.d/openstack-dashboard.conf
```

- Khởi động lại Dịch vụ:

```
systemctl restart httpd.service memcached.service
```

### 11. Cài đặt và cấu hình Cinder

Mô hình sử dụng một phân vùng riêng để lưu các volume của máy ảo.

![](../images/1-install-manual-ops/lsblk-controller.png)

- Tạo database cinder:

```
mysql -u root -p013279227Anh
CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
  IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
  IDENTIFIED BY '013279227Anh';
exit
```

- 

```
openstack user create --domain default --password 013279227Anh cinder
openstack role add --project service --user cinder admin
openstack service create --name cinderv2 --description "OpenStack Block Storage" volumev2
openstack service create --name cinderv3 --description "OpenStack Block Storage" volumev3
```

```
openstack endpoint create --region RegionOne volumev2 public http://172.16.2.56:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne volumev2 internal http://172.16.2.56:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne volumev2 admin http://172.16.2.56:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne volumev3 public http://172.16.2.56:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne volumev3 internal http://172.16.2.56:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne volumev3 admin http://172.16.2.56:8776/v3/%\(project_id\)s
```

- Tải packages:

```
yum install openstack-cinder targetcli python-keystone -y
```

- Cài đặt và cấu hình cinder volume với LVM:

```
yum install -y lvm2
```

- Khởi động dịch vụ LVM và cho phép khởi động cùng hệ thống.

```
systemctl enable lvm2-lvmetad.service
systemctl start lvm2-lvmetad.service
```

- Tạo LVM physical volume /dev/vdb

```
pvcreate /dev/vdb
```

- Tạo LVM volume group `cinder-volumes`

```
vgcreate cinder-volumes /dev/vdb
```

- Sửa file `/etc/lvm/lvm.conf`, để LVM chỉ scan ổ sdb cho block storage

```
devices {
...
filter = [ "a/sdb/", "r/.*/"]
```

- Sửa cấu hình cinder

```
cp /etc/cinder/cinder.conf /etc/cinder/cinder.conf.bak 
rm -rf /etc/cinder/cinder.conf
```

```
cat << EOF >> /etc/cinder/cinder.conf
[DEFAULT]
my_ip = 172.16.2.56
transport_url = rabbit://openstack:013279227Anh@172.16.2.56:5672
auth_strategy = keystone
osapi_volume_listen = 172.16.2.56
enabled_backends = lvm
[backend]
[backend_defaults]
[barbican]
[brcd_fabric_example]
[cisco_fabric_example]
[coordination]
[cors]
[database]
connection = mysql+pymysql://cinder:013279227Anh@172.16.2.56/cinder
[fc-zone-manager]
[healthcheck]
[key_manager]
[keystone_authtoken]
auth_uri = http://172.16.2.56:5000
auth_url = http://172.16.2.56:35357
memcached_servers = 172.16.2.56:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = 013279227Anh
[matchmaker_redis]
[nova]
[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues = true
rabbit_ha_queues = true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[oslo_reports]
[oslo_versionedobjects]
[profiler]
[service_user]
[ssl]
[vault]
[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
iscsi_protocol = iscsi
iscsi_helper = lioadm
volume_backend_name = lvm
EOF
```

- Phân quyền file cấu hình `/etc/cinder/cinder.conf`:

```
chown root:cinder /etc/cinder/cinder.conf
```

- Đồng bộ Database:

```
su -s /bin/sh -c "cinder-manage db sync" cinder
```

- Chỉnh sửa file `/etc/nova/nova.conf`

```
[cinder]
os_region_name = RegionOne
```

- Restart dịch vụ nova api

```
systemctl restart openstack-nova-api.service
```

- Enable va start dịch vụ:

```
systemctl enable openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service
systemctl restart openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service
```

## Phần 3. Cấu hình trên Node Compute

Thực hiện trên 2 node `compute01` và `compute02`

### 1. Cấu hình cơ bản

- Cấu hình cơ bản:

```
hostnamectl set-hostname compute01

sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl disable NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl enable network
sudo systemctl start network
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

- Cấu hình các mode sysctl

```
echo 'net.ipv4.conf.all.arp_ignore = 1'  >> /etc/sysctl.conf
echo 'net.ipv4.conf.all.arp_announce = 2'  >> /etc/sysctl.conf
echo 'net.ipv4.conf.all.rp_filter = 2'  >> /etc/sysctl.conf
echo 'net.netfilter.nf_conntrack_tcp_be_liberal = 1'  >> /etc/sysctl.conf
```

```
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

```
sysctl -p
```

- Update OS:

```
yum -y update
```

- Khai báo file hosts:

```
echo "172.16.2.56 controller" >> /etc/hosts
echo "172.16.2.59 compute01" >> /etc/hosts
echo "172.16.2.62 compute02" >> /etc/hosts
```

- Cài đặt các gói cần thiết:

```
yum -y install centos-release-openstack-queens
yum -y install crudini wget vim
yum -y install python-openstackclient openstack-selinux python2-PyMySQL
```

- Cài đặt và cấu hình NTP:

```
yum -y install chrony
VIP_MGNT_IP='172.16.2.56'
sed -i '/server/d' /etc/chrony.conf
echo "server $VIP_MGNT_IP iburst" >> /etc/chrony.conf
systemctl enable chronyd.service
systemctl restart chronyd.service
chronyc sources
```

![](../images/1-install-manual-ops/chrony.png)

- Chinh sua file `/etc/yum.repos.d/CentOS-QEMU-EV.repo`

### 2. Cài đặt nova

- Cài đặt Packages:

```
yum install openstack-nova-compute libvirt-client -y
```

- Cấu hình nova:

```
cp /etc/nova/nova.conf  /etc/nova/nova.conf.org
rm -rf /etc/nova/nova.conf
```

```
cat << EOF >> /etc/nova/nova.conf 
[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:013279227Anh@172.16.2.56:5672
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[api]
auth_strategy = keystone
[api_database]
[barbican]
[cache]
[cells]
[cinder]
[compute]
[conductor]
[console]
[consoleauth]
[cors]
[crypto]
[database]
[devices]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://172.16.2.56:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_url = http://172.16.2.56:5000/v3
memcached_servers = 172.16.2.56:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 013279227Anh
[libvirt]
virt_type = qemu
[matchmaker_redis]
[metrics]
[mks]
[neutron]
url = http://172.16.2.56:9696
auth_url = http://172.16.2.56:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = 013279227Anh
[notifications]
[osapi_v21]
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
rabbit_ha_queues = true
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues= true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[pci]
[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://172.16.2.56:5000/v3
username = placement
password = 013279227Anh
[quota]
[rdp]
[remote_debug]
[scheduler]
discover_hosts_in_cells_interval = 300
[serial_console]
[service_user]
[spice]
[upgrade_levels]
[vault]
[vendordata_dynamic_auth]
[vmware]
[vnc]
enabled = True
server_listen = 0.0.0.0
server_proxyclient_address = 172.16.2.59 // IP node compute tuong ung
novncproxy_base_url = http://172.16.2.56:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
EOF
```

- Phân quyền file cấu hình `/etc/nova/nova.conf`:

```
chown root:nova /etc/nova/nova.conf
```

### 3.  Cài đặt neutron

- Cài đặt Packages:

```
yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables -y
```

- Cấu hình neutron:

```
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org 
rm -rf /etc/neutron/neutron.conf
```

- Cấu hình file LB agent:

Lưu ý khi chạy đoạn ở dưới chú ý 2 tham số:

physical_interface_mappings = provider:ens256 (interface name provider)

local_ip = 10.10.41.72 (IP dải DATAVM `compute01`) - đổi sang IP tương ứng với `compute02`

```
cat << EOF >> /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:ens256
[network_log]
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = true
local_ip = 10.10.41.72
l2_population = true
EOF
```

- Cấu hình dhcp agent:

```
cp /etc/neutron/dhcp_agent.ini /etc/neutron/dhcp_agent.ini.org
rm -rf /etc/neutron/dhcp_agent.ini
```

```
cat << EOF >> /etc/neutron/dhcp_agent.ini
[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
force_metadata = True
[agent]
[ovs]
EOF
```

- Cấu hình metadata agent:

```
cp /etc/neutron/metadata_agent.ini /etc/neutron/metadata_agent.ini.org 
rm -rf /etc/neutron/metadata_agent.ini
```

```
cat << EOF >> /etc/neutron/metadata_agent.ini
[DEFAULT]
nova_metadata_host = 172.16.2.56
metadata_proxy_shared_secret = 013279227Anh
[agent]
[cache]
EOF
```

- Phân quyền

```
chown root:neutron /etc/neutron/metadata_agent.ini /etc/neutron/neutron.conf /etc/neutron/dhcp_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```

- Restart service nova

```
systemctl restart libvirtd.service openstack-nova-compute
```

- Enable, start service nova

```
systemctl enable neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
systemctl restart neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
```

## Phần 4. Một số thao tác trên Horizon

### 1. Truy cập Horizon

Truy cập theo `http://172.16.2.56/` sẽ tự động chuyển hướng sang `http://172.16.2.56/dashboard`

![](../images/1-install-manual-ops/horizen-login.png)

![](../images/1-install-manual-ops/horizon-login-success.png)

### 2. Tạo flavor

```
openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 0 m1.nano
openstack flavor create --id 1 --vcpus 1 --ram 1024 --disk 0 m1.tiny
openstack flavor create --id 2 --vcpus 2 --ram 2408 --disk 0 m1.small
```

![](../images/1-install-manual-ops/create-flavors.png)