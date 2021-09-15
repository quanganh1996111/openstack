# Cài đặt OpenStack High Availability - cấu hình node Compute

## Phần 1. Cấu hình ban đầu

- Đặt Hosts:

```
echo "172.16.3.20 controller1" >> /etc/hosts
echo "172.16.3.21 controller2" >> /etc/hosts
echo "172.16.3.22 controller3" >> /etc/hosts
echo "172.16.3.23 compute1" >> /etc/hosts
echo "172.16.3.24 compute2" >> /etc/hosts
```

- Tắt `Firewalld`, `Network Manager`, `SElinux`:

```
sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl disable NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl enable network
sudo systemctl start network
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

- Cài đặt `chrony`:

```
yum install chrony -y 

systemctl start chronyd 
systemctl enable chronyd
systemctl restart chronyd 

chronyc sources -v
```

```
sudo date -s "$(wget -qSO- --max-redirect=0 google.com 2>&1 | grep Date: | cut -d' ' -f5-8)Z"
ln -f -s /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime
```

- Chuẩn bị sysctl:

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

sysctl -p
```

- Reboot:

```
init 6
```

## Phần 2. Cài đặt OpenStack cho node compute

### 2.1 Cài đặt các gói cần thiết

```
yum -y install centos-release-openstack-queens
yum -y install crudini wget vim
yum -y install python-openstackclient openstack-selinux python2-PyMySQL
```

### 2.2. Cài đặt Nova

- Cài đặt gói

```
yum install openstack-nova-compute libvirt-client -y
```

- Cấu hình nova. Thay đổi IP VIP, IP node controller và IP node compute tương ứng.

```
cp /etc/nova/nova.conf  /etc/nova/nova.conf.org
rm -rf /etc/nova/nova.conf

cat << EOF >> /etc/nova/nova.conf 
[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:013279227Anh@172.16.3.20:5672,openstack:013279227Anh@172.16.3.21:5672,openstack:013279227Anh@172.16.3.22:5672
my_ip = 172.16.3.23
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
virt_type = qemu
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
server_proxyclient_address = 172.16.3.23
novncproxy_base_url = http://172.16.3.29:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
EOF
```

- Khởi động dịch vụ

```
chown root:nova /etc/nova/nova.conf
systemctl enable libvirtd.service openstack-nova-compute.service
systemctl restart libvirtd.service openstack-nova-compute.service
```

- Kiểm tra trên node `controller1`:

```
[root@controller1 ~]# openstack compute service list
+----+------------------+-------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host        | Zone     | Status  | State | Updated At                 |
+----+------------------+-------------+----------+---------+-------+----------------------------+
|  3 | nova-consoleauth | controller1 | internal | enabled | up    | 2021-09-15T09:58:10.000000 |
|  6 | nova-scheduler   | controller1 | internal | enabled | up    | 2021-09-15T09:58:12.000000 |
|  9 | nova-conductor   | controller1 | internal | enabled | up    | 2021-09-15T09:58:09.000000 |
| 27 | nova-conductor   | controller2 | internal | enabled | up    | 2021-09-15T09:58:12.000000 |
| 30 | nova-scheduler   | controller2 | internal | enabled | up    | 2021-09-15T09:58:13.000000 |
| 33 | nova-consoleauth | controller2 | internal | enabled | up    | 2021-09-15T09:58:13.000000 |
| 48 | nova-scheduler   | controller3 | internal | enabled | up    | 2021-09-15T09:58:04.000000 |
| 51 | nova-conductor   | controller3 | internal | enabled | up    | 2021-09-15T09:58:12.000000 |
| 54 | nova-consoleauth | controller3 | internal | enabled | up    | 2021-09-15T09:58:03.000000 |
| 63 | nova-compute     | compute1    | nova     | enabled | up    | None                       |
| 66 | nova-compute     | compute2    | nova     | enabled | up    | None                       |
+----+------------------+-------------+----------+---------+-------+----------------------------+
```

### 2.3. Cài đặt Neutron

- Cài đặt gói:

```
yum install openstack-neutron openstack-neutron-ml2 \
openstack-neutron-linuxbridge ebtables -y
```

- Cấu hình neutron:

```
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org 
rm -rf /etc/neutron/neutron.conf

cat << EOF >> /etc/neutron/neutron.conf
[DEFAULT]
transport_url = rabbit://openstack:013279227Anh@172.16.3.20:5672,openstack:013279227Anh@172.16.3.21:5672,openstack:013279227Anh@172.16.3.22:5672
auth_strategy = keystone
[agent]
[cors]
[database]
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

- Cấu hình LB Agent với IP tương ứng của từng node compute:

```
cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini.org 
rm -rf /etc/neutron/plugins/ml2/linuxbridge_agent.ini

cat << EOF >> /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[DEFAULT]
[agent]
extensions = qos
[linux_bridge]
physical_interface_mappings = provider:eth1 
[network_log]
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = true
local_ip = 10.10.40.23
l2_population = true
EOF
```

- Cấu hình DHCP Agent:

```
cp /etc/neutron/dhcp_agent.ini /etc/neutron/dhcp_agent.ini.org
rm -rf /etc/neutron/dhcp_agent.ini

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

cat << EOF >> /etc/neutron/metadata_agent.ini
[DEFAULT]
nova_metadata_host = 172.16.3.29
metadata_proxy_shared_secret = 013279227Anh
[agent]
[cache]
EOF
```

- Thêm vào file `/etc/nova/nova.conf`

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
```

- Phân quyền:

```
chown root:neutron /etc/neutron/metadata_agent.ini /etc/neutron/neutron.conf /etc/neutron/dhcp_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```

- Khởi động lại dịch vụ nova-compute

```
systemctl restart libvirtd.service openstack-nova-compute
```

- Khởi động dịch vụ

```
systemctl enable neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
systemctl restart neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
```

Sau bước này về node `controller1` thực hiện `openstack network agent list`

```
[root@controller1 ~]# openstack network agent list
+--------------------------------------+--------------------+-------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host        | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+-------------+-------------------+-------+-------+---------------------------+
| 2882e4f2-028f-4b9c-b0f3-05bfab78bc3e | Linux bridge agent | controller3 | None              | :-)   | UP    | neutron-linuxbridge-agent |
| 39f1a3ea-ef5a-426c-b790-1b75d7255380 | Linux bridge agent | controller2 | None              | :-)   | UP    | neutron-linuxbridge-agent |
| 81d230eb-10f3-4c53-9cf3-458c3456e702 | Metadata agent     | compute2    | None              | :-)   | UP    | neutron-metadata-agent    |
| 9703ae49-44b4-496a-b735-331ba0fbaa5f | Metadata agent     | compute1    | None              | :-)   | UP    | neutron-metadata-agent    |
| 97f1f32d-33cc-442b-ab35-fd4e4d937b65 | Linux bridge agent | compute1    | None              | :-)   | UP    | neutron-linuxbridge-agent |
| 9e666d90-e648-470d-83b4-af26e26c7457 | DHCP agent         | compute1    | nova              | :-)   | UP    | neutron-dhcp-agent        |
| bfeba981-9699-4db0-b73a-b2211e6dcc30 | Linux bridge agent | compute2    | None              | :-)   | UP    | neutron-linuxbridge-agent |
| c5aff5a2-f839-4606-9f1c-0352e3d29496 | DHCP agent         | compute2    | nova              | :-)   | UP    | neutron-dhcp-agent        |
| d4828fd8-7102-4ea5-9753-af10dc1b58eb | Linux bridge agent | controller1 | None              | :-)   | UP    | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+-------------+-------------------+-------+-------+---------------------------+
```