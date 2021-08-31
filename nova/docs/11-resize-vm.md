# Resize instance

Ta có thể thay đổi cấu hình của một instance bằng cách thay đổi flavor của nó.

## Cú pháp

openstack server resize --flavor FLAVOR SERVER

## 1. Resize máy ảo boot từ volume

- List danh sách VM

```
[root@controller1 ~]# openstack server list
+--------------------------------------+---------------+--------+-----------------------+-------------+---------+
| ID                                   | Name          | Status | Networks              | Image       | Flavor  |
+--------------------------------------+---------------+--------+-----------------------+-------------+---------+
| ef84a284-ba8d-4980-b888-ea852d313dd3 | VM5           | ACTIVE | provider=10.10.40.103 | cirros-ceph | m1.nano |
| e142cfbd-4417-4a21-adb3-2a21bd12254e | Provider_VM02 | ACTIVE | provider=10.10.40.113 | cirros-ceph | m1.nano |
| e8eb7a37-1c90-4b1e-85c9-ec14fe3b6501 | Provider_VM01 | ACTIVE | provider=10.10.40.114 | cirros-ceph | m1.nano |
+--------------------------------------+---------------+--------+-----------------------+-------------+---------+
```

- List các flavor

```
[root@controller1 ~]# openstack flavor list
+--------------------------------------+-----------+------+------+-----------+-------+-----------+
| ID                                   | Name      |  RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+-----------+------+------+-----------+-------+-----------+
| 0                                    | m1.nano   |   64 |    0 |         0 |     1 | True      |
| 1                                    | m1.tiny   | 1024 |    0 |         0 |     1 | True      |
| 4dfbfd2c-1f32-4ec0-8510-7bd1dab93b9d | m1.abcxyz |  256 |    0 |         0 |     1 | True      |
+--------------------------------------+-----------+------+------+-----------+-------+-----------+
```

- Resize VM `Provider_VM01` từ flavor `m1.nano` sang flavor `m1.abcxyz`

```
openstack server resize --flavor m1.abcxyz Provider_VM01
```

- Kiểm tra trạng thái VM:

```
openstack server list
+--------------------------------------+---------------+---------------+-----------------------+-------------+-----------+
| ID                                   | Name          | Status        | Networks              | Image       | Flavor    |
+--------------------------------------+---------------+---------------+-----------------------+-------------+-----------+
| ef84a284-ba8d-4980-b888-ea852d313dd3 | VM5           | ACTIVE        | provider=10.10.40.103 | cirros-ceph | m1.nano   |
| e142cfbd-4417-4a21-adb3-2a21bd12254e | Provider_VM02 | ACTIVE        | provider=10.10.40.113 | cirros-ceph | m1.nano   |
| e8eb7a37-1c90-4b1e-85c9-ec14fe3b6501 | Provider_VM01 | VERIFY_RESIZE | provider=10.10.40.114 | cirros-ceph | m1.abcxyz |
+--------------------------------------+---------------+---------------+-----------------------+-------------+-----------+
```

- Khi thấy trạng thái là `VERIFY_RESIZE`, tiến hành confirm:

```
openstack server resize --confirm Provider_VM01
```

- Nếu muốn hủy bỏ resize ta sử dụng lệnh:

```
openstack server resize --revert Provider_VM01
```

## 2. Resize máy ảo boot từ local

Thực hiện resize máy ảo bằng câu lệnh

```
openstack server resize --flavor <flavor-name> <vm-name>
```

Đợi đến khi trạng thái máy ảo chuyển về `VERIFY_RESIZE` (dùng câu lệnh `openstack server show` để xem). Ta tiến hành xác nhận hoặc loại bỏ kết quả của việc resize máy ảo. Tiến hành xác nhận bằng câu lệnh sau:

```
openstack server resize --confirm <vm-name>
```

Nếu muốn quay trở về sử dụng máy ảo cũ, sử dụng câu lệnh sau

```
openstack server resize --revert <vm-name>
```

**Lưu ý:** Đối với máy ảo boot từ local thì disk của flavor mới phải lớn hơn disk của flavor cũ