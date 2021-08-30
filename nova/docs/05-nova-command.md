# Các lệnh thường dùng trong Nova

## 1. Flavor

### List Flavor

```
[root@controller1 ~]# openstack flavor list
+----+----------+------+------+-----------+-------+-----------+
| ID | Name     |  RAM | Disk | Ephemeral | VCPUs | Is Public |
+----+----------+------+------+-----------+-------+-----------+
| 0  | m1.nano  |   64 |    0 |         0 |     1 | True      |
| 1  | m1.tiny  | 1024 |    0 |         0 |     1 | True      |
| 2  | m1.small | 2408 |    0 |         0 |     2 | True      |
+----+----------+------+------+-----------+-------+-----------+
```

### Tạo mới 1 flavor

```
openstack flavor create
    [--id <id>]
    [--ram <size-mb>]
    [--disk <size-gb>]
    [--ephemeral-disk <size-gb>]
    [--swap <size-mb>]
    [--vcpus <num-cpu>]
    [--rxtx-factor <factor>]
    [--public | --private]
    [--property <key=value> [...] ]
    [--project <project>]
    [--project-domain <project-domain>]
    <flavor-name>
```

Ví dụ:

```
openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 m1.nano
```

### Show chi tiết một flavor

```
[root@controller1 ~]# openstack flavor show m1.nano
+----------------------------+---------+
| Field                      | Value   |
+----------------------------+---------+
| OS-FLV-DISABLED:disabled   | False   |
| OS-FLV-EXT-DATA:ephemeral  | 0       |
| access_project_ids         | None    |
| disk                       | 0       |
| id                         | 0       |
| name                       | m1.nano |
| os-flavor-access:is_public | True    |
| properties                 |         |
| ram                        | 64      |
| rxtx_factor                | 1.0     |
| swap                       |         |
| vcpus                      | 1       |
+----------------------------+---------+
```

### Xóa flavor

```
openstack flavor delete <tên hoặc ID của flavor>
```

Ví dụ:

```
openstack flavor delete m1.small
```

### Update thuộc tính flavor

```
openstack flavor set
    [--no-property]
    [--property <key=value> [...] ]
    [--project <project>]
    [--project-domain <project-domain>]
    <flavor>
```

`--no-property` : Xóa tất cả các thuộc tính

### Bỏ một thuộc tính

```
openstack flavor unset
    [--property <key> [...] ]
    [--project <project>]
    [--project-domain <project-domain>]
    <flavor>
```

## 2. Server (VM)

### Tạo VM từ image hoặc volume

```
openstack server create
    (--image <image> | --volume <volume>)
    --flavor <flavor>
    [--security-group <security-group>]
    [--key-name <key-name>]
    [--property <key=value>]
    [--file <dest-filename=source-filename>]
    [--user-data <user-data>]
    [--availability-zone <zone-name>]
    [--block-device-mapping <dev-name=mapping>]
    [--nic <net-id=net-uuid,v4-fixed-ip=ip-addr,v6-fixed-ip=ip-addr,port-id=port-uuid,auto,none>]
    [--network <network>]
    [--port <port>]
    [--hint <key=value>]
    [--config-drive <config-drive-volume>|True]
    [--min <count>]
    [--max <count>]
    [--wait]
    <server-name>
```

Ví dụ:

```
openstack server create Provider_VM01 --flavor m1.nano --image cirros-0.3.4-x86_64-disk.img --nic net-id=de8ba495-8433-4043-8dac-cbffba6eb423 --security-group default
```

### List Server

```
openstack server list
```

```
+--------------------------------------+---------------+--------+-----------------------+-------------+---------+
| ID                                   | Name          | Status | Networks              | Image       | Flavor  |
+--------------------------------------+---------------+--------+-----------------------+-------------+---------+
| ef84a284-ba8d-4980-b888-ea852d313dd3 | VM5           | ACTIVE | provider=10.10.40.103 | cirros-ceph | m1.nano |
| e142cfbd-4417-4a21-adb3-2a21bd12254e | Provider_VM02 | ACTIVE | provider=10.10.40.113 | cirros-ceph | m1.nano |
| e8eb7a37-1c90-4b1e-85c9-ec14fe3b6501 | Provider_VM01 | ACTIVE | provider=10.10.40.114 | cirros-ceph | m1.nano |
+--------------------------------------+---------------+--------+-----------------------+-------------+---------+
```