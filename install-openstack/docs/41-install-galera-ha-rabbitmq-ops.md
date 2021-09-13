# Cài đặt High Availability - Rabbit MQ

## Phần 1. Cấu hình Galera MariaDB

Cấu hình trên 3 Node Controller

### 1. Cấu hình ban đầu

Thực hiện trên 3 node Controller (bước này có thể thao tác trên 2 node Compute luôn):

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

- Update OS:

```
yum install epel-release -y
yum update -y
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

- Reboot lại Server:

```
init 6
```

### 2. Cài đặt MariaDB

- Khai báo Repo để cài đặt:

```
echo '[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1' >> /etc/yum.repos.d/MariaDB.repo
yum -y update
```

- Cài đặt MariaDB:

```
yum install -y mariadb mariadb-server
```

- Cài đặt Galera và các gói hỗ trợ:

```
yum install -y galera rsync
```

- Stop MariaDB. **Lưu ý:** Không khởi động dịch vụ mariadb sau khi cài (Liên quan tới cấu hình Galera Mariadb)

### 3. Cấu hình Galera Cluster

- Node 1:

```
echo '[server]
[mysqld]
bind-address=172.16.3.20

[galera]
wsrep_on=ON
wsrep_provider=/usr/lib64/galera/libgalera_smm.so
#add your node ips here
wsrep_cluster_address="gcomm://10.10.41.20,10.10.41.21,10.10.41.22"
binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
#Cluster name
wsrep_cluster_name="db_cluster"
# Allow server to accept connections on all interfaces.
bind-address=172.16.3.20
# this server ip, change for each server
wsrep_node_address="10.10.41.20"
# this server name, change for each server
wsrep_node_name="node1"
wsrep_sst_method=rsync
[embedded]
[mariadb]
[mariadb-10.2]
' > /etc/my.cnf.d/server.cnf
```

### Node 2 và Node 3

Cấu hình tương ứng Node 1, Lưu ý thay thế các tham số theo đúng IP và Hostname của Node đó.

### Khởi động lại MariaDB

- Tại Node 1:

```
galera_new_cluster
systemctl start mariadb
systemctl enable mariadb
```

- Tại Node 2 và Node 3:

```
systemctl start mariadb
systemctl enable mariadb
```