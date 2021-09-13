# Cài đặt Multi Region cho Open Stack trên CentOS 7

## 1. Mô hình triển khai

Gồm 2 Hệ thống OpenStack chạy độc lập

## 2. IP Planning

![](../images/3-install-multi-region-ops/ip-planning.png)

## 3. Cài đặt 2 cụm Openstack độc lập

Lựa chọn cài đặt theo 2 cách:

- Cài Manual [tại đây](https://github.com/quanganh1996111/openstack/blob/main/install-openstack/docs/1-install-openstack-manual.md)

- Cài bằng Script [tại đây](https://github.com/quanganh1996111/openstack/blob/main/install-openstack/docs/2-install-openstack-script.md)

## 4. Cấu hình Region

Mặc định khi cài xong một cụm Openstack hệ thống sẽ nhận đó là **Region One**, ở bài này sẽ cho Cụm `172.16.2.72` là **Region One** và sửa cấu hình của Cụm `172.16.3.72` trỏ về **Region One**.

Cụm `172.16.3.72` gọi là **Region Two** sẽ chia sẻ 2 Dịch vụ `keystone`, `horizon` với `RegionOne`.

2 cụm sử dụng chung database `keystone` ở **Region One**, các service `glance`, `neutron`, `nova`, `cinder` ở node nào chạy database ở node đó.

### 4.1. Thực hiện trên Node Controller ở Region One

#### Tạo mới Region

**Lưu ý: Không source admin-openrc khi chạy keystone-manage** (thoát phiên ssh hiện tại ra và ssh lại phiên mới chạy lệnh tạo region)

```
keystone-manage bootstrap --bootstrap-password 013279227Anh \
--bootstrap-admin-url http://172.16.2.72:5000/v3/ \
--bootstrap-internal-url http://172.16.2.72:5000/v3/ \
--bootstrap-public-url http://172.16.2.72:5000/v3/ \
--bootstrap-region-id RegionTwo
```

- Kiểm tra các Region:

```
source admin-openrc
openstack region list
```

![](../images/3-install-multi-region-ops/create-region-two.png)

- Sau khi sử dụng keystone khởi tạo RegionTwo, keystone trên RegionOne sẽ tự động tạo thêm endpoint identity mới cho việc xác thực.

```
openstack endpoint list --service identity
```

![](../images/3-install-multi-region-ops/end-point-region-two.png)

#### Khởi tạo endpoint RegionTwo cho `nova`, `cinder`, `glance`, `neutron`

Thực hiện trên node `Controller` của **Region One**

Các endpoint tạo cho Region Two sẽ sử dụng IP `Controller` là `172.16.3.72`

```
openstack endpoint create --region RegionTwo image public http://172.16.3.72:9292
openstack endpoint create --region RegionTwo image admin http://172.16.3.72:9292
openstack endpoint create --region RegionTwo image internal http://172.16.3.72:9292

openstack endpoint create --region RegionTwo network public http://172.16.3.72:9696
openstack endpoint create --region RegionTwo network internal http://172.16.3.72:9696
openstack endpoint create --region RegionTwo network admin http://172.16.3.72:9696

openstack endpoint create --region RegionTwo compute public http://172.16.3.72:8774/v2.1
openstack endpoint create --region RegionTwo compute admin http://172.16.3.72:8774/v2.1
openstack endpoint create --region RegionTwo compute internal http://172.16.3.72:8774/v2.1

openstack endpoint create --region RegionTwo placement public http://172.16.3.72:8778
openstack endpoint create --region RegionTwo placement admin http://172.16.3.72:8778
openstack endpoint create --region RegionTwo placement internal http://172.16.3.72:8778

openstack endpoint create --region RegionTwo volumev2 public http://172.16.3.72:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionTwo volumev2 internal http://172.16.3.72:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionTwo volumev2 admin http://172.16.3.72:8776/v2/%\(project_id\)s

openstack endpoint create --region RegionTwo volumev3 public http://172.16.3.72:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionTwo volumev3 internal http://172.16.3.72:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionTwo volumev3 admin http://172.16.3.72:8776/v3/%\(project_id\)s
```

- Kiểm tra lại:

```
openstack endpoint list --region RegionTwo
```

![](../images/3-install-multi-region-ops/ops-endpoint-region-two-list.png)

### 4.2. Tạo admin openstack resource trên cho cả 2 node Controller 172.16.2.72 và 172.16.3.72

```
cat << EOF >> admin-openrc-r2
export OS_REGION_NAME=RegionTwo
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=013279227Anh
export OS_AUTH_URL=http://172.16.2.72:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export PS1='[\u@\h \W(admin-openrc)]\$ '
EOF
```

Hoặc:

```
export OS_REGION_NAME=RegionTwo
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=013279227Anh
export OS_AUTH_URL=http://172.16.2.72:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

## 5. Thực hiện config trên node Controller Region Two 172.16.3.72 để chỉnh sửa xác thực keystone node Controller Region One 172.16.2.72

Truy cập vào Node 172.16.3.72

### 5.1. Service glance

**Chỉnh sửa 2 file glance-api.conf và glance-registry.conf**

Mục `[keystone_authtoken]`

`auth_uri`, `auth_url` về Controller 172.16.2.72

Chỉnh sửa `region_name` về `RegionTwo`.

**Kết quả:**

- File `/etc/glance/glance-api.conf`

```
[root@controller-site1 ~]# cat /etc/glance/glance-api.conf | egrep -v '^$|^#'
[DEFAULT]
transport_url = rabbit://openstack:013279227Anh@172.16.3.72
[cors]
[database]
connection = mysql+pymysql://glance:013279227Anh@172.16.3.72/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[image_format]
[keystone_authtoken]
www_authenticate_uri = http://172.16.2.72:5000
auth_url = http://172.16.2.72:5000
memcached_servers = 172.16.3.72:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = 013279227Anh
region_name = RegionTwo
[matchmaker_redis]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
driver = messagingv2
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
```

- File `/etc/glance/glance-registry.conf`

```
[root@controller-site1 ~]# cat /etc/glance/glance-registry.conf | egrep -v '^$|^#'
[DEFAULT]
transport_url = rabbit://openstack:013279227Anh@172.16.3.72
[database]
connection = mysql+pymysql://glance:013279227Anh@172.16.3.72/glance
[keystone_authtoken]
www_authenticate_uri = http://172.16.2.72:5000
auth_url = http://172.16.2.72:5000
memcached_servers = 172.16.3.72:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = 013279227Anh
region_name = RegionTwo
[matchmaker_redis]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
driver = messagingv2
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
```

**Restart lại Dịch vụ**

```
systemctl restart openstack-glance-api.service openstack-glance-registry.service
```

**Kiểm tra trên controller 2 region với điều kiện xác thực `admin-openrc-r2`**

```
source admin-openrc-r2
openstack --debug image list --os-region-name RegionTwo
```

- Upload image mới lên để kiểm tra xác thực admin-openrc-r2 (Kiểm tra với `cirros`)

```
openstack image create "cirrosr2" --file cirros-0.3.5-x86_64-disk.img --disk-format qcow2 --container-format bare --public
```

- Đứng trên **Region One** chạy lệnh kiểm tra **Region Two**

```
openstack image list --os-region-name RegionTwo
```

![](../images/3-install-multi-region-ops/cirros-test.png)

**Lưu ý**: Đứng ở **Region** nào up thì chỉ xuất hiện image ở **Region** đó.

### 5.2. Service nova

Đối với service nova, cần chỉnh sửa lại config trên node ở **Region Two** bao gồm node `Controller` và các node `Compute`.

#### Thực hiện trên Node Controller 172.16.3.72

Chỉnh sửa lại mục các mục `[keystone_authtoken]`, `[neutron]`, `[placement]`, `[cinder]` tại `/etc/nova/nova.conf`

- Mục `[cinder]`: Chỉnh sửa `os_region_name` về `RegionTwo`

- Mục `[keystone_authtoken]`:

```
auth_url về 172.16.2.72
region_name về RegionTwo
```

- Mục `[neutron]`:

```
auth_url về 172.16.2.72
region_name về RegionTwo
```

- Mục `[placement]`:

```
os_region_name về RegionTwo
auth_url về 172.16.2.72
```

```
[root@controller1 ~]# cat /etc/nova/nova.conf | egrep -v "^#|^$"
[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:013279227Anh@172.16.3.72
my_ip = 172.16.3.72
use_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver
osapi_compute_listen = $my_ip
metadata_listen = $my_ip
instance_usage_audit = True
instance_usage_audit_period = hour
notify_on_state_change = vm_and_task_state
[api]
auth_strategy = keystone
[api_database]
connection = mysql+pymysql://nova:013279227Anh@172.16.3.72/nova_api
[barbican]
[cache]
[cells]
[cinder]
os_region_name = RegionTwo
[compute]
[conductor]
[console]
[consoleauth]
[cors]
[crypto]
[database]
connection = mysql+pymysql://nova:013279227Anh@172.16.3.72/nova
[devices]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://172.16.3.72:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_uri = http://172.16.2.72:5000
auth_url = http://172.16.2.72:5000
memcached_servers = 172.16.3.72:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = 013279227Anh
region_name = RegionTwo
[libvirt]
[matchmaker_redis]
[metrics]
[mks]
[neutron]
url = http://172.16.3.72:9696
auth_url = http://172.16.2.72:5000
auth_type = password
project_domain_name = Default
user_domain_name = Default
region_name = RegionTwo
project_name = service
username = neutron
password = 013279227Anh
service_metadata_proxy = True
metadata_proxy_shared_secret = 013279227Anh
[notifications]
[osapi_v21]
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
driver = messagingv2
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[pci]
[placement]
os_region_name = RegionTwo
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://172.16.2.72:5000/v3
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
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip
novncproxy_host = $my_ip
[workarounds]
[wsgi]
[xenserver]
[xvp]
```

- Restart lại service nova:

```
systemctl restart openstack-nova-api.service openstack-nova-scheduler.service openstack-nova-consoleauth.service openstack-nova-conductor.service openstack-nova-novncproxy.service
```

- Kiểm tra: Đứng trên node `Controller` **Regione One**

```
openstack --debug server list --os-region-name RegionTwo
```

```
openstack compute service list --os-region-name RegionTwo
```

#### Thực hiện trên các node Compute

Chỉnh sửa `/etc/nova/nova.conf`

- Mục `[cinder]`:

```
Chỉ định sử dụng cinder service của RegionTwo (os_region_name = RegionTwo)
```

- Mục `[keystone_authtoken]`:

```
Chỉnh sửa auth_url về 172.16.2.72
region_name về RegionTwo
```

- Mục `[neutron]`:

```
Chỉnh sửa auth_url về 172.16.2.72
region_name về RegionTwo
```

- Mục `[placement]`:

```
Chỉnh sửa auth_url về 172.16.2.72
Sửa os_region_name về RegionTwo
```

```
[root@compute1-site1 ~]# cat /etc/nova/nova.conf | egrep -v "^#|^$"
[DEFAULT]
enabled_apis = osapi_compute,metadata
my_ip = 172.16.3.73
use_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver
transport_url = rabbit://openstack:013279227Anh@172.16.3.72
instance_usage_audit = True
instance_usage_audit_period = hour
notify_on_state_change = vm_and_task_state
[api]
auth_strategy = keystone
[api_database]
[barbican]
[cache]
[cells]
[cinder]
os_region_name = RegionTwo
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
api_servers = http://172.16.3.72:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_uri = http://172.16.2.72:5000
auth_url = http://172.16.2.72:5000
memcached_servers = 172.16.3.72:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = 013279227Anh
region_name = RegionTwo
[libvirt]
virt_type = kvm
[matchmaker_redis]
[metrics]
[mks]
[neutron]
url = http://172.16.3.72:9696
auth_url = http://172.16.2.72:5000
auth_type = password
project_domain_name = Default
user_domain_name = Default
region_name = RegionTwo
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
driver = messagingv2
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[pci]
[placement]
os_region_name = RegionTwo
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://172.16.2.72:5000/v3
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
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://172.16.3.72:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
```

- Kiểm tra log tại Nova Compute

```
cat /var/log/nova/nova-compute.log | grep 'placement'
```

Nếu xuất hiện lỗi:

```
2021-08-18 22:13:52.187 5420 ERROR nova.scheduler.client.report [req-91929c78-1b99-4fa5-a80f-60f028af493a - - - - -] [req-fcd18e50-3416-4a90-8de3-45dec982a9b2] Failed to retrieve resource provider tree from placement API for UUID 00bbeb2b-0213-435c-b545-50e32232b21a. Got 401: {"error": {"message": "The request you have made requires authentication.", "code": 401, "title": "Unauthorized"}}.
```

Kiểm tra lại cấu hình Nova trên node `Controller` và các Node `Compute` của **Region Two**, sau đó khởi động lại OS các node (nguyên nhân do database hoặc cache)

- Thực hiện trên node `Controller` của **Region One**:

```
openstack compute service list --os-region-name RegionTwo
```

![](../images/3-install-multi-region-ops/nova-controller-compute.png)

### 5.3. Service cinder

**Thực hiện trên node Controller của Region Two**

Với dịch vụ Cinder, ta sẽ chỉnh sửa các mục `[keystone_authtoken]` tại `/etc/cinder/cinder.conf`

```
Sửa auth_uri, auth_url về 172.16.2.72
region_name về RegionTwo
```

- Restart service cinder:

```
systemctl restart openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service
```

- Kiểm tra trên Node `Controller` của **Region One**

```
openstack volume service list --os-region-name RegionTwo
```

![](../images/3-install-multi-region-ops/ops-volume-list.png)

### 5.4. Service neutron

Với Neutron, ta cần chỉnh sửa lại settings trên node `Controller` và các node `Compute` của **Region Two**

#### Thực hiện trên Node Controller 172.16.2.72

Chỉnh sửa các mục `[keystone_authtoken]`, `[nova]` tại /etc/neutron/neutron.conf

- Mục [keystone_authtoken]:

```
Sửa auth_uri, auth_url về 172.16.2.72
region_name về RegionTwo
```

Mục [nova]

auth_url về CTL 172.16.2.72
region_name = RegionTwo

#### Thực hiện trên các Node Compute

Chỉnh sửa lại các mục [keystone_authtoken] tại /etc/neutron/neutron.conf

```
Sửa auth_uri, auth_url về 172.16.2.72
region_name về RegionTwo
```

- Khởi động lại Dịch vụ:

```
systemctl restart neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
```

- Kiểm tra trên node `Controller` ở **Region One**

```
openstack network agent list --os-region-name RegionTwo
```

![](../images/3-install-multi-region-ops/network-agent-list.png)

### Kiểm tra trên Horizon

Truy cập vào Horizon Dashboard

![](../images/3-install-multi-region-ops/ops-two-region-horizon.png)


## 6. Một số lỗi 

- Xuất hiện lỗi tại Node Compute

```
[root@compute01 ~]# cat /var/log/nova/nova-compute.log | grep ERROR

2019-04-11 11:40:05.363 15299 ERROR nova.compute.manager [instance: 5350c69f-24de-4345-9556-0cc92faa3ef2] BuildAbortException: Build of instance 5350c69f-24de-4345-9556-0cc92faa3ef2 aborted: Invalid input received: Invalid image identifier or unable to access requested image. (HTTP 400) (Request-ID: req-1e69aa25-4f63-477d-a8a5-678ebf1bb869)
```

Kiểm tra lại cấu hình cinder:

Tại Controller, có thể thiếu `glance_api_servers` section `[glance_api_servers]` `(/etc/cinder/cinder.conf)`

Tại Compute, có thể thiếu `os_region_name` tại section `[cinder]` `(/etc/nova/nova.conf)`

- Nếu xuất hiện lỗi tại Node Compute

```
2021-08-18 22:13:52.187 5420 ERROR nova.compute.manager [instance: 66eef324-058d-443e-afa6-8893f183a7db] PortBindingFailed: Binding failed for port 68e62053-fed2-4bd8-b3a8-0755012774ad, please check neutron logs for more information.
```

Kiểm tra lại cấu hình `neutron` và `service`

- Lỗi khi tạo máy ảo

```
2021-08-18 22:13:52.187 5420 1296 ERROR nova.compute.manager ResourceProviderCreationFailed: Failed to create resource provider compute1
```

Chưa config mục `[placement]`:

Chỉnh sửa `auth_url` về CTL 172.16.2.72

```
Sửa `os_region_name` về `RegionTwo`
```

## 7. Redirect dashboard horizon cụm region 2 về horizon cụm region 1

Thao tác trên server horizon region `172.16.2.72`

```
vi /var/www/html/index.html
```

```
<html>
<head>
<META HTTP-EQUIV="Refresh" Content="0.5; URL=http://172.16.2.72/dashboard">
</head>
<body>
<center> <h1>Redirecting to OpenStack Dashboard</h1> </center>
</body>
</html>
```

- Restart lại httpd

```
systemctl restart httpd.service
```