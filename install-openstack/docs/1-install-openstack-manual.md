# Cài đặt OpenStack Manual

## 1. Mô hình triển khai

Mô hình triển khai gồm 1 node Controller, 2 node Compute.

![](../images/1-install-manual-ops/install-manual-ops.png)

## 2. Danh sách IP

![](../images/1-install-manual-ops/IP-planning-manual.png)

## 3. Bật chế độ ảo hóa trên KVM

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