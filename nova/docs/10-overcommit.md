# Overcommit RAM & CPU

## Khái niệm
OpenStack cho phép bạn khai báo vượt quá CPU và RAM trên các node Compute. OpenStack cho phép bạn overcommit CPU và RAM trên node compute. Điều này cho phép tăng số instance chạy trên cloud của bạn.

Theo mặc định, Compute service cho phép tỉ lệ sau:

- **CPU** : 16:1
- **RAM** : 1.5:1

Công thức tính số lượng các máy ảo trên compute node là **(OR*PC)/VC**, ở đây:

- **OR**: CPU overcommit ratio (virtual cores tương ứng với mỗi physical core)
- **PC**: Số physical cores.
- **VC**: Số virtual cores mỗi instance.

## Cấu hình
Cấu hình chỉ số ratio cho CPU và RAM ở file `/etc/nova/nova.conf` trên node **Controller**.

```
[DEFAULT]
cpu_allocation_ratio = 0.0
ram_allocation_ratio = 0.0
disk_allocation_ratio = 0.0
```

Nếu ta để ratio là 0.0 thì chỉ số ratio mặc định của cpu, ram, disk là `16.0`, `1.5`, `1.0`

Sau khi cấu hình, restart lại dịch vụ của nova:
```
systemctl restart openstack-nova-api.service
systemctl restart openstack-nova-scheduler.service
systemctl restart openstack-nova-novncproxy.service
systemctl restart openstack-nova-conductor.service
```