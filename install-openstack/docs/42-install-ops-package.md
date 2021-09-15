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