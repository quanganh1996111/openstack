# Cài đặt OpenStack Manual

## Mục lục

## Phần 1. Chuẩn bị

### 1. Mô hình triển khai

Mô hình triển khai gồm 1 node Controller, 2 node Compute.

![](../images/1-install-manual-ops/install-manual-ops.png)

### 2. Danh sách IP

![](../images/1-install-manual-ops/IP-planning-manual.png)

### 3. Bật chế độ ảo hóa trên KVM

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

### 2.1. Cấu hình cơ bản

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

### 2.2. Cài đặt và cấu hình NTP

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

### 2.3. Cài đặt và cấu hình memcache

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

### 2.4. Cài đặt và cấu hình MariaDB

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

### 2.5. Cài đặt và cấu hình RabbitMQ

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

### 2.6. Cài đặt keystone

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

