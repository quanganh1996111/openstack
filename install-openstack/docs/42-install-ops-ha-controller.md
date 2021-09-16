# Cài đặt OpenStack High Availability

Sau khi hoàn tất cài đặt Galera và RabbitMQ ở bài trước. Ta tiến hành cài đặt OpenStack

## Phần 4. Cài đặt các gói cần thiết cho OPS (Trên tất cả các node)

- Cài đặt các gói cần thiết:

```
yum -y install centos-release-openstack-queens
yum -y install crudini wget vim
yum -y install python-openstackclient openstack-selinux python2-PyMySQL
```

- Cài đặt Memcache

**Lưu ý:** Cấu hình tương ứng với IP của node controller và lần lượng từ `controller1` đến `controller2` và `controller3`

```
yum install -y memcached

sed -i "s/-l 127.0.0.1,::1/-l 172.16.3.20/g" /etc/sysconfig/memcached

systemctl enable memcached.service
systemctl restart memcached.service
```

## Phần 5. Cài đặt Keystone

### 5.1 Tạo Database

Thực hiện trên node `controller1`

```
mysql -u root -p013279227Anh
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY '013279227Anh';

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'controller1' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'controller2' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'controller3' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
exit
```

### 5.2. Cài đặt các gói cần thiết

#### Thực hiện trên tất cả các node:

- Cài đặt các gói cần thiết

```
yum install openstack-keystone httpd mod_wsgi -y
```

- Cấu hình bind port keystone:

**Lưu ý:** đổi IP tương ứng với IP của các node Controller

```
cp /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
sed -i -e 's/VirtualHost \*/VirtualHost 172.16.3.20/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/Listen 5000/Listen 172.16.3.20:5000/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/Listen 35357/Listen 172.16.3.20:35357/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/^Listen.*/Listen 172.16.3.20:80/g' /etc/httpd/conf/httpd.conf
```

- Cấu hình dịch vụ:

```
cp /etc/keystone/keystone.conf /etc/keystone/keystone.conf.org
rm -rf /etc/keystone/keystone.conf

cat << EOF >> /etc/keystone/keystone.conf
[DEFAULT]
[assignment]
[auth]
[cache]
[catalog]
[cors]
[credential]
[database]
connection = mysql+pymysql://keystone:013279227Anh@172.16.3.29/keystone
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
#driver = messagingv2
[oslo_messaging_rabbit]
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues = true
rabbit_ha_queues = true
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

- Phân quyền:

```
chown root:keystone /etc/keystone/keystone.conf
```

- Đồng bộ Database:

```
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

- Setup Fernet key:

```
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

- Chuyển 2 thư mục này sang 2 node controller còn lại:

```
scp -r /etc/keystone/credential-keys /etc/keystone/fernet-keys root@controller2:/etc/keystone/
scp -r /etc/keystone/credential-keys /etc/keystone/fernet-keys root@controller3:/etc/keystone/
```

- Bootstrap:

```
keystone-manage bootstrap --bootstrap-password 013279227Anh \
--bootstrap-admin-url http://172.16.3.20:5000/v3/ \
--bootstrap-internal-url http://172.16.3.20:5000/v3/ \
--bootstrap-public-url http://172.16.3.20:5000/v3/ \
--bootstrap-region-id RegionOne
```

- Enable và khởi động httpd:

```
systemctl enable httpd.service
systemctl start httpd.service
```

- Export biến môi trường:

```
export OS_USERNAME=admin
export OS_PASSWORD=013279227Anh
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://172.16.3.29:35357/v3
export OS_IDENTITY_API_VERSION=3
```

- Tạo domain, user:

```
openstack domain create --description "An Example Domain" example
openstack project create --domain default \
--description "Service Project" service

openstack project create --domain default \
--description "Demo Project" demo

openstack user create --domain default \
--password 013279227Anh demo

openstack role create user
openstack role add --project demo --user demo user

unset OS_AUTH_URL OS_PASSWORD

openstack --os-auth-url http://172.16.3.29:35357/v3 \
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
export OS_AUTH_URL=http://172.16.3.29:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF

cat << EOF >> demo-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=013279227Anh
export OS_AUTH_URL=http://172.16.3.29:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```

#### Tiến hành cấu hình Keystone trên node controller2 và controller3

```
cp /etc/keystone/keystone.conf /etc/keystone/keystone.conf.org
rm -rf /etc/keystone/keystone.conf

cat << EOF >> /etc/keystone/keystone.conf
[DEFAULT]
[assignment]
[auth]
[cache]
[catalog]
[cors]
[credential]
[database]
connection = mysql+pymysql://keystone:013279227Anh@172.16.3.29/keystone
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
#driver = messagingv2
[oslo_messaging_rabbit]
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues = true
rabbit_ha_queues = true
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

- Phân quyền:

```
chown root:keystone /etc/keystone/keystone.conf
chown -R keystone:keystone /etc/keystone/credential-keys /etc/keystone/fernet-keys
```

- Enable và bật httpd:

```
systemctl enable httpd.service
systemctl start httpd.service
```

#### Kiểm tra 

Test theo test case sau:

- Bật HTTPD tại CTL 1, tắt HTTP trên CTL 2 3, lấy token

- Bật HTTPD tại CTL 2, tắt HTTP trên CTL 1 3, lấy token

- Bật HTTPD tại CTL 3, tắt HTTP trên CTL 1 2, lấy token

Bảo đảm các test case đều pass

```
systemctl stop httpd
source admin-openrc
openstack token issue
```

Kiểm tra đồng thời trạng thái trên HA Proxy Stats: http://172.16.3.29:8080/stats

![](../images/4-install-ops-ha-ceph/Screenshot_5.png)

## Phần 6. Cài đặt Glance

### Thực hiện trên node Controller1

- Tạo Database:

```
mysql -u root -p013279227Anh

CREATE DATABASE glance;

GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
IDENTIFIED BY '013279227Anh';

GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
IDENTIFIED BY '013279227Anh';

GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'controller1' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'controller2' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'controller3' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
exit
```

- Tạo User:

```
openstack user create --domain default --password 013279227Anh glance
openstack role add --project service --user glance admin
openstack service create --name glance \
--description "OpenStack Image" image
```

- Tạo endpoint:

```
openstack endpoint create --region RegionOne \
image public http://172.16.3.29:9292
openstack endpoint create --region RegionOne \
image admin http://172.16.3.29:9292
openstack endpoint create --region RegionOne \
image internal http://172.16.3.29:9292
```

### Thực hiện trên tất cả các node controller

- Cài packages:

```
yum install -y openstack-glance
```

- Cấu hình glance-api. Thay đổi IP tương ứng với IP VIP và IP của node controller đang cấu hình:

```
cat << EOF >> /etc/glance/glance-api.conf
[DEFAULT]
bind_host = 172.16.3.20
registry_host = 172.16.3.29
[cors]
[database]
connection = mysql+pymysql://glance:013279227Anh@172.16.3.29/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[image_format]
[keystone_authtoken]
auth_uri = http://172.16.3.29:5000
auth_url = http://172.16.3.29:5000
memcached_servers = 172.16.3.20:11211,172.16.3.21:11211,10.10.11.89:11211
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
#driver = messagingv2
[oslo_messaging_rabbit]
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues = true
rabbit_ha_queues = true
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

- Cấu hình glance-registry. Thay đổi IP tương ứng với IP VIP và IP của node controller đang cấu hình:

```
cat << EOF >> /etc/glance/glance-registry.conf
[DEFAULT]
bind_host = 172.16.3.20
[database]
connection = mysql+pymysql://glance:013279227Anh@172.16.3.29/glance
[keystone_authtoken]
auth_uri = http://172.16.3.29:5000
auth_url = http://172.16.3.29:5000
memcached_servers = 172.16.3.20:11211,172.16.3.21:11211,172.16.3.22:11211
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
#driver = messagingv2
[oslo_messaging_rabbit]
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues = true
rabbit_ha_queues = true
[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
EOF
```

- Phân quyền:

```
chown root:glance /etc/glance/glance-api.conf
chown root:glance /etc/glance/glance-registry.conf
```

- Sync Database. Đứng trên node `controller1` để tiến hành sync database.

```
su -s /bin/sh -c "glance-manage db_sync" glance
```

- Enable và start dịch vụ:

```
systemctl enable openstack-glance-api.service \
openstack-glance-registry.service

systemctl start openstack-glance-api.service \
openstack-glance-registry.service
```

- Download và tạo image:

```
wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
openstack image create "cirros" \
--file cirros-0.3.5-x86_64-disk.img \
--disk-format qcow2 --container-format bare \
--public
```

**Lưu ý:** Sau khi tạo images, mặc định image sẽ được đưa vào thư mục /var/lib/glance/images trên 1 trong 3 node, ta cần scp image này sang 2 node còn lại vào chính thư mục đó. Đồng thơi phân quyền.

Ví dụ: image `6f7e9132-682f-4f74-aa1c-4ec7471f7740` vừa khởi tạo bên trên đang nằm ở node `controller3`. Ta cần phải copy và phân quyền lại.

```
[root@controller3 ~]# ls /var/lib/glance/images
6f7e9132-682f-4f74-aa1c-4ec7471f7740

scp /var/lib/glance/images/6f7e9132-682f-4f74-aa1c-4ec7471f7740 root@controller1:/var/lib/glance/images/
scp /var/lib/glance/images/6f7e9132-682f-4f74-aa1c-4ec7471f7740 root@controller2:/var/lib/glance/images/
```

```
chown -R glance:glance /var/lib/glance/images
```

### Kiểm tra lại cài đặt.

Test theo test case sau:

- Bật Glance services tại CTL 1, tắt Glance services trên CTL 2 3, get list image

- Bật Glance services tại CTL 2, tắt Glance services trên CTL 1 3, get list image

- Bật Glance services tại CTL 3, tắt Glance services trên CTL 1 2, get list image

Bảo đảm các test case đều pass:

```
systemctl stop openstack-glance-api.service \
openstack-glance-registry.service
openstack image list
```

Kiểm tra đồng thời trạng thái trên HA Proxy Stats: http://172.16.3.29:8080/stats

![](../images/4-install-ops-ha-ceph/Screenshot_6.png)

## Phần 7. Cài đặt Nova

### Thực hiện trên tất cả Controller

- Cài đặt gói:

```
yum install -y openstack-nova-api openstack-nova-conductor \
openstack-nova-console openstack-nova-novncproxy \
openstack-nova-scheduler openstack-nova-placement-api
```

### Cấu hình trên node controller1

- Tạo Database trên node `controller1`:

```
mysql -u root -p013279227Anh

CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;

GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'controller1' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'controller2' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'controller3' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;


GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'controller1' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'controller2' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'controller3' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;


GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \
IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \
IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'controller1' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'controller2' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'controller3' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
exit
```

- Tạo user và endpoint:

```
openstack user create --domain default --password 013279227Anh nova
openstack role add --project service --user nova admin
openstack service create --name nova \
--description "OpenStack Compute" compute

openstack endpoint create --region RegionOne \
compute public http://172.16.3.29:8774/v2.1
openstack endpoint create --region RegionOne \
compute admin http://172.16.3.29:8774/v2.1
openstack endpoint create --region RegionOne \
compute internal http://172.16.3.29:8774/v2.1

openstack user create --domain default --password 013279227Anh placement
openstack role add --project service --user placement admin
openstack service create --name placement --description "Placement API" placement
  
openstack endpoint create --region RegionOne placement public http://172.16.3.29:8778
openstack endpoint create --region RegionOne placement admin http://172.16.3.29:8778
openstack endpoint create --region RegionOne placement internal http://172.16.3.29:8778
```

### Cấu hình nova trên tất cả các node controller

**Lưu ý:** Yêu cầu thực hiện xong bước tạo endpoint trên node `controller1`. Cấu hình IP VIP và IP tương ứng trên từng node controller.

```
cp /etc/nova/nova.conf /etc/nova/nova.conf.org
rm -rf /etc/nova/nova.conf

cat << EOF >> /etc/nova/nova.conf
[DEFAULT]
my_ip = 172.16.3.20
enabled_apis = osapi_compute,metadata
use_neutron = True
osapi_compute_listen=172.16.3.20
metadata_host=172.16.3.20
metadata_listen=172.16.3.20
metadata_listen_port=8775
firewall_driver = nova.virt.firewall.NoopFirewallDriver
notify_on_state_change = vm_and_task_state
transport_url = rabbit://openstack:013279227Anh@172.16.3.20:5672,openstack:013279227Anh@172.16.3.21:5672,openstack:013279227Anh@172.16.3.22:5672
[api]
auth_strategy = keystone
[api_database]
connection = mysql+pymysql://nova:013279227Anh@172.16.3.29/nova_api
[barbican]
[cache]
backend = oslo_cache.memcache_pool
enabled = true
memcache_servers = 172.16.3.20:11211,172.16.3.21:11211,172.16.3.22:11211
[cells]
[cinder]
[compute]
[conductor]
[console]
[consoleauth]
[cors]
[crypto]
[database]
connection = mysql+pymysql://nova:013279227Anh@172.16.3.29/nova
[devices]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://172.16.3.29:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_url = http://172.16.3.29:5000/v3
memcached_servers = 172.16.3.20:11211,172.16.3.21:11211,172.16.3.22:11211
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
#driver = messagingv2
[oslo_messaging_rabbit]
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues = true
rabbit_ha_queues = true
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
auth_url = http://172.16.3.29:5000/v3
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
novncproxy_host=172.16.3.20
enabled = true
#vncserver_listen = 172.16.3.20
#vncserver_proxyclient_address = 172.16.3.20
novncproxy_base_url = http://172.16.3.29:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
EOF
```

- Thêm vào file `00-nova-placement-api.conf`

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

- Cấu hình bind port cho nova-placement với IP tương ứng của từng node controller:

```
sed -i -e 's/VirtualHost \*/VirtualHost 172.16.3.20/g' /etc/httpd/conf.d/00-nova-placement-api.conf
sed -i -e 's/Listen 8778/Listen 172.16.3.20:8778/g' /etc/httpd/conf.d/00-nova-placement-api.conf

systemctl restart httpd
```

### Sync Database trên node controller1

```
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
su -s /bin/sh -c "nova-manage db sync" nova
```

- Enable và start nova service trên các tất cả các node controller:

```
systemctl enable openstack-nova-api.service \
openstack-nova-scheduler.service openstack-nova-consoleauth.service \
openstack-nova-conductor.service openstack-nova-novncproxy.service

systemctl restart openstack-nova-api.service \
openstack-nova-scheduler.service openstack-nova-consoleauth.service \
openstack-nova-conductor.service openstack-nova-novncproxy.service
```

```
[root@controller1 ~]# openstack compute service list
+----+------------------+-------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host        | Zone     | Status  | State | Updated At                 |
+----+------------------+-------------+----------+---------+-------+----------------------------+
|  3 | nova-consoleauth | controller1 | internal | enabled | up    | 2021-09-15T07:21:27.000000 |
|  6 | nova-scheduler   | controller1 | internal | enabled | up    | 2021-09-15T07:21:27.000000 |
|  9 | nova-conductor   | controller1 | internal | enabled | up    | 2021-09-15T07:21:27.000000 |
| 27 | nova-conductor   | controller2 | internal | enabled | up    | 2021-09-15T07:21:28.000000 |
| 30 | nova-scheduler   | controller2 | internal | enabled | up    | 2021-09-15T07:21:28.000000 |
| 33 | nova-consoleauth | controller2 | internal | enabled | up    | 2021-09-15T07:21:29.000000 |
| 48 | nova-scheduler   | controller3 | internal | enabled | up    | 2021-09-15T07:21:29.000000 |
| 51 | nova-conductor   | controller3 | internal | enabled | up    | 2021-09-15T07:21:29.000000 |
| 54 | nova-consoleauth | controller3 | internal | enabled | up    | 2021-09-15T07:21:29.000000 |
+----+------------------+-------------+----------+---------+-------+----------------------------+
```

### Kiểm tra

Test theo các use case:

- Bật Nova services tại CTL 1, tắt Nova services trên CTL 2 3, list nova services

- Bật Nova services tại CTL 2, tắt Nova services trên CTL 1 3, list nova services

- Bật Nova services tại CTL 3, tắt Nova services trên CTL 1 2, list nova services

Bảo đảm các test case đều pass

```
systemctl stop openstack-nova-api.service \
openstack-nova-scheduler.service openstack-nova-consoleauth.service \
openstack-nova-conductor.service openstack-nova-novncproxy.service

openstack compute service list
```

Kiểm tra đồng thời trạng thái trên HA Proxy Stats: http://172.16.3.29:8080/stats

![](../images/4-install-ops-ha-ceph/Screenshot_7.png)

## Phần 8. Cài đặt Neutron

### Cài đặt Package trên tất cả các node controller

```
yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables -y
```

### Thao tác trên node controller1

- Tạo Database:

```
mysql -u root -p013279227Anh
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'controller1' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'controller2' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'controller3' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
exit
```

- Tạo user, endpoint:

```
openstack user create --domain default --password 013279227Anh neutron
openstack role add --project service --user neutron admin
openstack service create --name neutron \
--description "OpenStack Networking" network
  
openstack endpoint create --region RegionOne \
network public http://172.16.3.29:9696
openstack endpoint create --region RegionOne \
network internal http://172.16.3.29:9696
openstack endpoint create --region RegionOne \
network admin http://172.16.3.29:9696
```

### Thao tác cấu hình trên tất cả các node controller

**Lưu ý:** Cấu hình IP VIP và IP tương ứng với node controller.

- Cấu hình file neutron

```
cat << EOF >> /etc/neutron/neutron.conf
[DEFAULT]
bind_host = 172.16.3.20
core_plugin = ml2
transport_url = rabbit://openstack:013279227Anh@172.16.3.20:5672,openstack:013279227Anh@172.16.3.21:5672,openstack:013279227Anh@172.16.3.22:5672
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
allow_overlapping_ips = True
dhcp_agents_per_network = 2
service_plugins = neutron.services.qos.qos_plugin.QoSPlugin

[agent]
[cors]
[database]
connection = mysql+pymysql://neutron:013279227Anh@172.16.3.29/neutron
[keystone_authtoken]
auth_uri = http://172.16.3.29:5000
auth_url = http://172.16.3.29:35357
memcached_servers = 172.16.3.20:11211,172.16.3.21:11211,172.16.3.22:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = 013279227Anh
[matchmaker_redis]
[nova]
auth_url = http://172.16.3.29:35357
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
#driver = messagingv2
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

cat << EOF >> /etc/neutron/plugins/ml2/ml2_conf.ini
[DEFAULT]
[l2pop]
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security,qos
[ml2_type_flat]
flat_networks = provider
[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]
#network_vlan_ranges = provider
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

- Cấu hình trên file l3 agent:

```
cp /etc/neutron/l3_agent.ini /etc/neutron/l3_agent.ini.org
rm -rf /etc/neutron/l3_agent.ini

cat << EOF >> /etc/neutron/l3_agent.ini
[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
[agent]
[ovs]
EOF
```

- Chỉnh sửa file `/etc/nova/nova.conf`. Thêm cấu hình:

```
[neutron]
url = http://172.16.3.29:9696
auth_url = http://172.16.3.29:35357
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

### Thực hiện trên node controller1

- Restart dịch vụ nova:

```
systemctl restart openstack-nova-api.service openstack-nova-scheduler.service openstack-nova-consoleauth.service openstack-nova-conductor.service openstack-nova-novncproxy.service
```

- Phân quyền:

```
chown -R root:neutron /etc/neutron/
```

- Tạo liên kết:

```
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

- Sync Database:

```
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

- Enable và start dịch vụ:

```
systemctl enable neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service
systemctl restart neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service
```

### Thao tác trên các node controller2 và controller3

- Restart dịch vụ nova:

```
systemctl restart openstack-nova-api.service openstack-nova-scheduler.service openstack-nova-consoleauth.service openstack-nova-conductor.service openstack-nova-novncproxy.service
```

- Phân quyền:

```
chown -R root:neutron /etc/neutron/
```

- Tạo liên kết:

```
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

- Sync Database:

```
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

- Enable và start dịch vụ:

```
systemctl enable neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service
systemctl restart neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service
```

### Kiểm tra

Test lại dịch vụ:

- Bật Neutron service tại CTL 1, tắt Neutron service trên CTL 2 3, list Neutron service

- Bật Neutron service tại CTL 2, tắt Neutron service trên CTL 1 3, list Neutron service

- Bật Neutron service tại CTL 3, tắt Neutron service trên CTL 1 2, list Neutron service

```
systemctl stop neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service
openstack network agent list
```

Kiểm tra đồng thời trạng thái trên HA Proxy Stats: http://172.16.3.29:8080/stats

## Phần 9. Cài đặt Horizon

### Thực hiện trên tất cả các node controller

- Cài đặt package:

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
<META HTTP-EQUIV="Refresh" Content="0.5; URL=http://172.16.3.29/dashboard">
</head>
<body>
<center> <h1>Redirecting to OpenStack Dashboard</h1> </center>
</body>
</html>
EOF
```

### Thực hiện trên node controller1

- Cấu hình file Horizon:

```
cp /etc/openstack-dashboard/local_settings /etc/openstack-dashboard/local_settings.org
```

```
vi /etc/openstack-dashboard/local_settings

### Thay đổi cấu hình trong file /etc/openstack-dashboard/local_settings
ALLOWED_HOSTS = ['*',]
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'Default'

### Lưu ý thêm SESSION_ENGINE vào trên dòng CACHE như bên dưới
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': ['172.16.3.20:11211','172.16.3.21:11211','172.16.3.22:11211',]
    }
}
OPENSTACK_HOST = "172.16.3.29"
OPENSTACK_KEYSTONE_URL = "http://172.16.3.29:5000/v3"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"

### Lưu ý: Nếu chỉ sử dụng provider, chỉnh sửa các thông số sau
OPENSTACK_NEUTRON_NETWORK = {
    'enable_router': False,
    'enable_quotas': False,
    'enable_ipv6': False,
    'enable_distributed_router': False,
    'enable_ha_router': False,
    'enable_fip_topology_check': False,
}

TIME_ZONE = "Asia/Ho_Chi_Minh"
```

- Scp sang 2 node controller còn lại

**Lưu ý:** Yêu cầu đã thực hiện xong bước xóa cấu hình dashboard trên 2 node còn lại

```
scp /etc/openstack-dashboard/local_settings root@controller2:/etc/openstack-dashboard/
scp /etc/openstack-dashboard/local_settings root@controller3:/etc/openstack-dashboard/
```

- Phân quyền lại cho file vừa copy:

```
chown root:apache /etc/openstack-dashboard/local_settings
```

- Thêm vào file `/etc/httpd/conf.d/openstack-dashboard.conf` cho tất cả các node controller:

```
echo "WSGIApplicationGroup %{GLOBAL}" >> /etc/httpd/conf.d/openstack-dashboard.conf
```

- Restart lại httpd, memcached

```
systemctl restart httpd.service memcached.service
```

### Kiểm tra

Đăng nhập vào Horizon qua IP VIP: http://172.16.3.29/

![](../images/4-install-ops-ha-ceph/Screenshot_8.png)

### Sau bước này tiến hành cấu hình các node compute

## Phần 10. Cấu hình Cinder

Sau khi hoàn thành cấu hình cho các node compute, quay lại cấu hình Cinder cho các node Controller. Để cấu hình High Availability cho OpenStack ta cần CEPH.

Tham khảo việc cài đặt CEPH [tại đây](https://github.com/quanganh1996111/ceph/tree/main/ceph/thuc-hanh/docs)

- Tải package cho tất cả các node controller

```
yum install openstack-cinder -y
```

### Thao tác trên node controller1

- Tạo Database:

```
mysql -u root -p013279227Anh
CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
IDENTIFIED BY '013279227Anh';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'controller1' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'controller2' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'controller3' IDENTIFIED BY '013279227Anh';FLUSH PRIVILEGES;
exit
```

- Tạo mới endpoint:

```
openstack user create --domain default --password 013279227Anh cinder
openstack role add --project service --user cinder admin
openstack service create --name cinderv2 \
--description "OpenStack Block Storage" volumev2
  
openstack service create --name cinderv3 \
--description "OpenStack Block Storage" volumev3
  
openstack endpoint create --region RegionOne \
volumev2 public http://172.16.3.29:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne \
volumev2 internal http://172.16.3.29:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne \
volumev2 admin http://172.16.3.29:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne \
volumev3 public http://172.16.3.29:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne \
volumev3 internal http://172.16.3.29:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne \
volumev3 admin http://172.16.3.29:8776/v3/%\(project_id\)s
```

### Chỉnh sửa cấu hình cinder trên tất cả các node controller

Cấu hình IP VIP, IP tương ứng với các node controller

```
cp /etc/cinder/cinder.conf /etc/cinder/cinder.conf.bak
rm -rf /etc/cinder/cinder.conf

cat << EOF >> /etc/cinder/cinder.conf
[DEFAULT]
my_ip = 172.16.3.20
transport_url = rabbit://openstack:013279227Anh@172.16.3.20:5672,openstack:013279227Anh@172.16.3.21:5672,openstack:013279227Anh@172.16.3.22:5672
auth_strategy = keystone
osapi_volume_listen = 172.16.3.20
[backend]
[backend_defaults]
[barbican]
[brcd_fabric_example]
[cisco_fabric_example]
[coordination]
[cors]
[database]
connection = mysql+pymysql://cinder:013279227Anh@172.16.3.29/cinder
[fc-zone-manager]
[healthcheck]
[key_manager]
[keystone_authtoken]
auth_uri = http://172.16.3.29:5000
auth_url = http://172.16.3.29:35357
memcached_servers = 172.16.3.20:11211,172.16.3.21:11211,172.16.3.22:11211
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
#driver = messagingv2
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
EOF
```

- Phân quyền:

```
chown root:cinder /etc/cinder/cinder.conf
```

- Chỉnh sửa file `/etc/nova/nova.conf`

```
[cinder]
os_region_name = RegionOne
```

### Quay trở lại node controller1 để đồng bộ Database

```
su -s /bin/sh -c "cinder-manage db sync" cinder
```

### Restart lại Dịch vụ trên tất cả các node controller

- Restart lại dịch vụ nova api

```
systemctl restart openstack-nova-api.service
```

- Enable và start dịch vụ cinder:

```
systemctl enable openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service
systemctl start openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service
```

- Kiểm tra lại Dịch vụ:

```
[root@controller1 ~]# openstack volume service list
+------------------+-------------+------+---------+-------+----------------------------+
| Binary           | Host        | Zone | Status  | State | Updated At                 |
+------------------+-------------+------+---------+-------+----------------------------+
| cinder-scheduler | controller1 | nova | enabled | up    | 2021-09-16T01:27:33.000000 |
| cinder-scheduler | controller2 | nova | enabled | up    | 2021-09-16T01:27:36.000000 |
| cinder-scheduler | controller3 | nova | enabled | up    | 2021-09-16T01:27:39.000000 |
+------------------+-------------+------+---------+-------+----------------------------+
```

**Lưu ý:** Tại thời điểm chưa chỉ định cinder backend lý do sẽ tích hợp Ceph làm backend chung cho Cinder

### Kiểm tra

Test theo test case sau:

- Bật cinder tại CTL 1, tắt cinder trên CTL 2 3, lấy list service

- Bật cinder tại CTL 2, tắt cinder trên CTL 1 3, lấy list service

- Bật cinder tại CTL 3, tắt cinder trên CTL 1 2, lấy list service

- Sau khi cấu hình Cinder tại CTL 1 2 3

```
systemctl stop openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service
openstack volume service list
```

Kiểm tra đồng thời HAProxy Stats: http://172.16.3.29:8080/stats