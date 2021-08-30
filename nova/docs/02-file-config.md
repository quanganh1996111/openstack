# File cấu hình của Nova

File cấu hình Nova đặt tại `/etc/nova/nova.conf`

## 1. File cấu hình trên node Controller

### [DEFAULT]

- Ip managerent của node Controller:

```
my_ip = x.x.x.x
```

- Hỗ trợ neutron service:

```
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
```

### [api]

- Xác thực qua Keystone

```
auth_strategy = keystone
```

### [api_database]

```
connection = mysql+pymysql://nova:013279227Anh@172.16.3.24/nova_api
```

### [cache]

- Cấu hình cache sử dụng `memcache`:

```
backend = oslo_cache.memcache_pool
enabled = true
memcache_servers = 172.16.3.24:11211
```

### [database]

- Cấu hình database của nova-service:

```
connection = mysql+pymysql://nova:013279227Anh@172.16.3.24/nova
```

### [glance]

- Đường dẫn dịch vụ glance:

```
api_servers = http://172.16.3.24:9292
```

### [keystone_authtoken]

- URL xác thực với keystone:

```
auth_url = http://172.16.3.24:5000/v3
```

- Đường dẫn memcache server:

```
memcached_servers = 172.16.3.24:11211
```

- Kiểu xác thực:

```
auth_type = password
```

- Thông tin domain:

```
project_domain_name = default
user_domain_name = default
```

- Project name của Nova:

```
project_name = service
```

- Thông tin xác thực:

```
username = nova
password = 013279227Anh
```

- Region:

```
region_name = RegionOne
```

### [neutron]

- Các cấu hình liên quan tới neutron-service:

```
url = http://172.16.3.24:9696
auth_url = http://172.16.3.24:35357
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

### [placement]

- các thông tin xác thực, project với placement:

```
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://172.16.3.24:5000/v3
username = placement
password = 013279227Anh
```

### [vnc]

- Địa chỉ management IP của VNC proxy:

```
novncproxy_host=172.16.3.24
enabled = true
vncserver_listen = 172.16.3.24
vncserver_proxyclient_address = 172.16.3.24
novncproxy_base_url = http://172.16.3.24:6080/vnc_auto.html
```

## 2. File cấu hình của Nova-compute

### [libvirt]

Loại ảo hóa sử dụng: Có thể là `kvm`, `lxc`, `qemu`, `uml`, `xen`, `parallels`

```
images_rbd_pool=vms
images_type=rbd
rbd_secret_uuid=44b453eb-f1ae-45c3-8d1e-30b0cf795c06
rbd_user=nova
images_rbd_ceph_conf = /etc/ceph/ceph.conf
virt_type = qemu
```

**Lưu ý:** Các thông số `images_rbd_pool`, `images_type`, `rbd_secret_uuid`, `rbd_user`, `images_rbd_ceph_conf` là các cấu hình tích hợp CEPH với OpenStack. Tham khảo [tại đây](https://github.com/quanganh1996111/ceph/blob/main/ceph/thuc-hanh/docs/5-install-ceph-openstack.md)