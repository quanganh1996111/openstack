# Khái niệm Endpoint và Catalog

## Phần 1. Endpoint

- Endpoint trong Keystone là một URL có thể được sử dụng để truy cập dịch vụ trong Openstack.

- 1 Endpoint giống như một điểm liên lạc để người dùng sử dụng dịch vụ của Openstack.

### 3 loại Endpoint

- `adminurl` : Dùng cho người dùng quản trị.

- `internalurl` : Là những gì các dịch vụ khác sử dụng để giao tiếp với nhau.

- `publicurl` : Là những gì mà người khác truy cập vào endpoint sử dụng dịch vụ.

### Show list endpoint

```
openstack endpoint list
```

```
+----------------------------------+-----------+--------------+--------------+---------+-----------+--------------------------------------------+
| ID                               | Region    | Service Name | Service Type | Enabled | Interface | URL                                        |
+----------------------------------+-----------+--------------+--------------+---------+-----------+--------------------------------------------+
| 056f317a7cd140e082aa30b8115c95dd | RegionOne | nova         | compute      | True    | public    | http://172.16.2.72:8774/v2.1/%(tenant_id)s |
| 07ffc79e1b5a489e9336475ddfc6a8b9 | RegionOne | keystone     | identity     | True    | internal  | http://172.16.2.72:5000/v3/                |
| 15cf6d13f06741428365f04b4174efbe | RegionOne | keystone     | identity     | True    | public    | http://172.16.2.72:5000/v3/                |
| 25a42da02f4643768c1b120f26f897c8 | RegionTwo | nova         | compute      | True    | public    | http://172.16.3.72:8774/v2.1               |
| 28092dde24194d31851112734beb2f94 | RegionTwo | glance       | image        | True    | internal  | http://172.16.3.72:9292                    |
| 2b696b917bc548ceab016d55f9f11d81 | RegionTwo | glance       | image        | True    | public    | http://172.16.3.72:9292                    |
| 2d18f2a133644809835ab84b22844eea | RegionTwo | cinderv2     | volumev2     | True    | admin     | http://172.16.3.72:8776/v2/%(project_id)s  |
| 2d235dee8bc34ea08376e8c4ff4c97c6 | RegionOne | cinderv2     | volumev2     | True    | internal  | http://172.16.2.72:8776/v2/%(tenant_id)s   |
| 2e0f1de503b1407cbf933abd7263df87 | RegionTwo | keystone     | identity     | True    | public    | http://172.16.2.72:5000/v3/                |
| 3593a1baf77c4e2f90002440d698b3a9 | RegionOne | placement    | placement    | True    | admin     | http://172.16.2.72:8778                    |
| 3a76a038068e4d679af799e42c421182 | RegionOne | glance       | image        | True    | internal  | http://172.16.2.72:9292                    |
| 3b05bf47e4fb485d857ad7fd0bef5d12 | RegionOne | cinderv3     | volumev3     | True    | public    | http://172.16.2.72:8776/v3/%(tenant_id)s   |
| 3f1fd2aab3d9448db8d01ac2ddcc99f4 | RegionTwo | cinderv3     | volumev3     | True    | admin     | http://172.16.3.72:8776/v3/%(project_id)s  |
| 44b5283e0090401bb30f41ba064820bb | RegionOne | cinderv2     | volumev2     | True    | public    | http://172.16.2.72:8776/v2/%(tenant_id)s   |
| 4ba1a2acf7b1407a8e08086d8079eecc | RegionTwo | placement    | placement    | True    | admin     | http://172.16.3.72:8778                    |
| 4c2b54d476024da3ac08e8a24a3dc555 | RegionTwo | nova         | compute      | True    | admin     | http://172.16.3.72:8774/v2.1               |
| 4cab4259f0d94477b18bc8a4554f996e | RegionTwo | keystone     | identity     | True    | internal  | http://172.16.2.72:5000/v3/                |
| 51ce257e8bc244279a9d738be7be3ed9 | RegionOne | placement    | placement    | True    | public    | http://172.16.2.72:8778                    |
| 586c122ad2074349895e7987a622d7bd | RegionOne | neutron      | network      | True    | admin     | http://172.16.2.72:9696                    |
| 58ca2bb600ed4914b892999738538c4c | RegionTwo | neutron      | network      | True    | admin     | http://172.16.3.72:9696                    |
| 6e6d5d86be474d8099ae4601b25b4137 | RegionTwo | neutron      | network      | True    | public    | http://172.16.3.72:9696                    |
| 6f8e08cf7df649c983dc0d97283e5661 | RegionTwo | cinderv3     | volumev3     | True    | public    | http://172.16.3.72:8776/v3/%(project_id)s  |
| 73451a13ad5b4f989e62279677e70329 | RegionOne | cinderv3     | volumev3     | True    | internal  | http://172.16.2.72:8776/v3/%(tenant_id)s   |
| 745f3e9b15a24e64b8aaf25e4041ef6c | RegionTwo | cinderv2     | volumev2     | True    | public    | http://172.16.3.72:8776/v2/%(project_id)s  |
| 75ff657956d8447a95a323ff43cf1585 | RegionTwo | nova         | compute      | True    | internal  | http://172.16.3.72:8774/v2.1               |
| 80c92be25fc0486ab2b127f4106cd805 | RegionOne | cinderv2     | volumev2     | True    | admin     | http://172.16.2.72:8776/v2/%(tenant_id)s   |
| 84c880d5fab740aab5c44ffa6a48cda7 | RegionOne | placement    | placement    | True    | internal  | http://172.16.2.72:8778                    |
| 88ca57e4de734ce18bcf26e179294a36 | RegionTwo | neutron      | network      | True    | internal  | http://172.16.3.72:9696                    |
| 8f83aa21b1504240b3f8380f8ff62db1 | RegionTwo | keystone     | identity     | True    | admin     | http://172.16.2.72:5000/v3/                |
| 92c66d4b4a4a4a2eaed494df8fcff366 | RegionOne | keystone     | identity     | True    | admin     | http://172.16.2.72:5000/v3/                |
| 93af54029efd4ca1a12457c4be45dac4 | RegionTwo | placement    | placement    | True    | internal  | http://172.16.3.72:8778                    |
| 9621785f9a4041f1a78570b849e51b03 | RegionOne | glance       | image        | True    | admin     | http://172.16.2.72:9292                    |
| 9a5ef17942d042e091cff4f70b6c701a | RegionOne | nova         | compute      | True    | admin     | http://172.16.2.72:8774/v2.1/%(tenant_id)s |
| a25a16c855274396b21e323d52a3a4ae | RegionTwo | placement    | placement    | True    | public    | http://172.16.3.72:8778                    |
| d4cc960d503a48ccb66069b3c8bc0d10 | RegionTwo | glance       | image        | True    | admin     | http://172.16.3.72:9292                    |
| d5cdcb1eb2964487bc99f1d619e2d54c | RegionOne | nova         | compute      | True    | internal  | http://172.16.2.72:8774/v2.1/%(tenant_id)s |
| d7b9ad90248e4d3e95cb51a67fd4e8f3 | RegionOne | neutron      | network      | True    | public    | http://172.16.2.72:9696                    |
| ddc7b3c027a34fcc85eaaba700347968 | RegionOne | glance       | image        | True    | public    | http://172.16.2.72:9292                    |
| df754531d10046eb81d6abc8ae0cb516 | RegionOne | neutron      | network      | True    | internal  | http://172.16.2.72:9696                    |
| e4db90c183da4a2f90d638961ed82704 | RegionTwo | cinderv2     | volumev2     | True    | internal  | http://172.16.3.72:8776/v2/%(project_id)s  |
| e8cb201a50ac4a88ad2da5ab5710aff1 | RegionTwo | cinderv3     | volumev3     | True    | internal  | http://172.16.3.72:8776/v3/%(project_id)s  |
| f29feab469eb41cdb95fe8082f557d0a | RegionOne | cinderv3     | volumev3     | True    | admin     | http://172.16.2.72:8776/v3/%(tenant_id)s   |
+----------------------------------+-----------+--------------+--------------+---------+-----------+--------------------------------------------
```

**Hiển thị chi tiết một endpoint:**

```
openstack endpoint show <endpoint>
```

Trong đó: `<endpoint>` có thể là `endpoint ID`, `service ID`, `service name`, `service type`

Ví dụ:

```
[root@controller1 ~]# openstack endpoint show a25a16c855274396b21e323d52a3a4ae
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | a25a16c855274396b21e323d52a3a4ae |
| interface    | public                           |
| region       | RegionTwo                        |
| region_id    | RegionTwo                        |
| service_id   | 2be3c01193fc49a784e4db8917d2e319 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://172.16.3.72:8778          |
+--------------+----------------------------------+
```

## Phần 2. Catalog

Catalog trong Openstack là tập hợp các danh mục sẵn sàng sử dụng mà khách hàng có thể sử dụng trong OPS.

Service catalog cung cấp cho người dùng về các dịch vụ có sẵn trong OPS, cùng với thông tin bổ sung về các vùng, phiên bản API và các project có sẵn.

Catalog làm cho việc tìm kiếm dịch vụ hiệu quả, chẳng hạn như cách định cấu hình liên lạc giữa các dịch vụ.

### List Catalog

```
openstack catalog list
```

### Show thông tin một mục trong catalog

```
openstack catalog show <service>
```

Trong đó: `<service>` là Dịch vụ và bạn muốn hiển thị.

Ví dụ:

```
[root@controller1 ~]# openstack catalog show keystone
+-----------+-----------------------------------------+
| Field     | Value                                   |
+-----------+-----------------------------------------+
| endpoints | RegionOne                               |
|           |   internal: http://172.16.2.72:5000/v3/ |
|           | RegionOne                               |
|           |   public: http://172.16.2.72:5000/v3/   |
|           | RegionTwo                               |
|           |   public: http://172.16.2.72:5000/v3/   |
|           | RegionTwo                               |
|           |   internal: http://172.16.2.72:5000/v3/ |
|           | RegionTwo                               |
|           |   admin: http://172.16.2.72:5000/v3/    |
|           | RegionOne                               |
|           |   admin: http://172.16.2.72:5000/v3/    |
|           |                                         |
| id        | d8dec9f1fec3489a95f5fbe530251c9f        |
| name      | keystone                                |
| type      | identity                                |
+-----------+-----------------------------------------+
```