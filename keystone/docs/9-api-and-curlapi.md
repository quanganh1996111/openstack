# Tìm hiểu về API và sử dụng lệnh curl để gọi API

## 1. API

### API là gì ?

- API : viết tắt của Application Programming Interface - giao diện lập trình ứng dụng.

- API là các phương thức, giao thức kết nối với các thư viện và ứng dụng khác.

- API cung cấp khả năng cung cấp khả năng truy xuất đến một tập các hàm hay dùng. Và từ đó có thể trao đổi dữ liệu giữa các ứng dụng.

## 2. Sử dụng lệnh curl để gọi API Openstack

### 2.1. Lấy Token

**Dùng curl**

**Method:** POST

```
curl -X POST -i \
-H "Content-Type: application/json" \
-d \
'{ "auth": {
    "identity": {
        "methods": ["password"],
        "password": {
            "user": {
              "name": "admin",
              "domain": { "name": "Default" },
              "password": "013279227Anh"
            }
          }
        },
        "scope": {
          "project": {
            "name": "admin",
            "domain": { "name": "Default" }
          }
        }
  }
}' \
http://localhost:5000/v3/auth/tokens
```

Option sử dụng trong lệnh curl:

- `-i (--include)`: Output sẽ chứa cả HTTP-header

- `-H (--header)`: kết hợp với Content-Type để xác định kiểu dữ liệu truyền vào header. Tại đây là dạng json

- `-d (--data)`: Dữ liệu truyền vào body

**Kết quả:**

```
HTTP/1.1 201 Created
Date: Fri, 20 Aug 2021 08:24:32 GMT
Server: Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5
X-Subject-Token: gAAAAABhH2bBGtedyswuIu0RCNnt44gvCoSeaM3krk72CfAHV4ec6G_OBHC1P810R9M13vTCcbIsJVp0Wj8ivsf2F_ShkWowWFINrQWDIoFuIt-JGI13avIIa3EqbWrAE8fKqj3fUUCbEfTNNPTc6pk_NzZiiPLb7o8m5JmChOBILwJtfLb9edw
Vary: X-Auth-Token
x-openstack-request-id: req-9d007fe8-ca9e-498e-b40d-2c049c5c09b8
Content-Length: 8119
Content-Type: application/json

{"token": {"is_domain": false, "methods": ["password"], "roles": [{"id": "3be3b4945c264f26b7e8936ea31dba10", "name": "admin"}], "expires_at": "2021-08-20T09:24:33.000000Z", "project": {"domain": {"id": "default", "name": "Default"}, "id": "a21e39dfc7f542cdb3d350a21165b5ca", "name": "admin"}, "catalog": [{"endpoints": [{"region_id": "RegionOne", "url": "http://172.16.2.72:8776/v3/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionOne", "interface": "public", "id": "3b05bf47e4fb485d857ad7fd0bef5d12"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:8776/v3/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionTwo", "interface": "admin", "id": "3f1fd2aab3d9448db8d01ac2ddcc99f4"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:8776/v3/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionTwo", "interface": "public", "id": "6f8e08cf7df649c983dc0d97283e5661"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:8776/v3/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionOne", "interface": "internal", "id": "73451a13ad5b4f989e62279677e70329"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:8776/v3/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionTwo", "interface": "internal", "id": "e8cb201a50ac4a88ad2da5ab5710aff1"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:8776/v3/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionOne", "interface": "admin", "id": "f29feab469eb41cdb95fe8082f557d0a"}], "type": "volumev3", "id": "1bff5297b2ea4591be0ee95b6639ce6e", "name": "cinderv3"}, {"endpoints": [{"region_id": "RegionOne", "url": "http://172.16.2.72:8778", "region": "RegionOne", "interface": "admin", "id": "3593a1baf77c4e2f90002440d698b3a9"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:8778", "region": "RegionTwo", "interface": "admin", "id": "4ba1a2acf7b1407a8e08086d8079eecc"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:8778", "region": "RegionOne", "interface": "public", "id": "51ce257e8bc244279a9d738be7be3ed9"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:8778", "region": "RegionOne", "interface": "internal", "id": "84c880d5fab740aab5c44ffa6a48cda7"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:8778", "region": "RegionTwo", "interface": "internal", "id": "93af54029efd4ca1a12457c4be45dac4"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:8778", "region": "RegionTwo", "interface": "public", "id": "a25a16c855274396b21e323d52a3a4ae"}], "type": "placement", "id": "2be3c01193fc49a784e4db8917d2e319", "name": "placement"}, {"endpoints": [{"region_id": "RegionTwo", "url": "http://172.16.3.72:8776/v2/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionTwo", "interface": "admin", "id": "2d18f2a133644809835ab84b22844eea"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:8776/v2/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionOne", "interface": "internal", "id": "2d235dee8bc34ea08376e8c4ff4c97c6"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:8776/v2/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionOne", "interface": "public", "id": "44b5283e0090401bb30f41ba064820bb"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:8776/v2/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionTwo", "interface": "public", "id": "745f3e9b15a24e64b8aaf25e4041ef6c"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:8776/v2/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionOne", "interface": "admin", "id": "80c92be25fc0486ab2b127f4106cd805"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:8776/v2/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionTwo", "interface": "internal", "id": "e4db90c183da4a2f90d638961ed82704"}], "type": "volumev2", "id": "5f066234d4d448cf991a37444751bb5f", "name": "cinderv2"}, {"endpoints": [{"region_id": "RegionOne", "url": "http://172.16.2.72:8774/v2.1/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionOne", "interface": "public", "id": "056f317a7cd140e082aa30b8115c95dd"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:8774/v2.1", "region": "RegionTwo", "interface": "public", "id": "25a42da02f4643768c1b120f26f897c8"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:8774/v2.1", "region": "RegionTwo", "interface": "admin", "id": "4c2b54d476024da3ac08e8a24a3dc555"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:8774/v2.1", "region": "RegionTwo", "interface": "internal", "id": "75ff657956d8447a95a323ff43cf1585"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:8774/v2.1/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionOne", "interface": "admin", "id": "9a5ef17942d042e091cff4f70b6c701a"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:8774/v2.1/a21e39dfc7f542cdb3d350a21165b5ca", "region": "RegionOne", "interface": "internal", "id": "d5cdcb1eb2964487bc99f1d619e2d54c"}], "type": "compute", "id": "cc8a0f7dbae14f5d9d6e7a1c584478d7", "name": "nova"}, {"endpoints": [{"region_id": "RegionOne", "url": "http://172.16.2.72:5000/v3/", "region": "RegionOne", "interface": "internal", "id": "07ffc79e1b5a489e9336475ddfc6a8b9"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:5000/v3/", "region": "RegionOne", "interface": "public", "id": "15cf6d13f06741428365f04b4174efbe"}, {"region_id": "RegionTwo", "url": "http://172.16.2.72:5000/v3/", "region": "RegionTwo", "interface": "public", "id": "2e0f1de503b1407cbf933abd7263df87"}, {"region_id": "RegionTwo", "url": "http://172.16.2.72:5000/v3/", "region": "RegionTwo", "interface": "internal", "id": "4cab4259f0d94477b18bc8a4554f996e"}, {"region_id": "RegionTwo", "url": "http://172.16.2.72:5000/v3/", "region": "RegionTwo", "interface": "admin", "id": "8f83aa21b1504240b3f8380f8ff62db1"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:5000/v3/", "region": "RegionOne", "interface": "admin", "id": "92c66d4b4a4a4a2eaed494df8fcff366"}], "type": "identity", "id": "d8dec9f1fec3489a95f5fbe530251c9f", "name": "keystone"}, {"endpoints": [{"region_id": "RegionTwo", "url": "http://172.16.3.72:9292", "region": "RegionTwo", "interface": "internal", "id": "28092dde24194d31851112734beb2f94"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:9292", "region": "RegionTwo", "interface": "public", "id": "2b696b917bc548ceab016d55f9f11d81"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:9292", "region": "RegionOne", "interface": "internal", "id": "3a76a038068e4d679af799e42c421182"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:9292", "region": "RegionOne", "interface": "admin", "id": "9621785f9a4041f1a78570b849e51b03"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:9292", "region": "RegionTwo", "interface": "admin", "id": "d4cc960d503a48ccb66069b3c8bc0d10"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:9292", "region": "RegionOne", "interface": "public", "id": "ddc7b3c027a34fcc85eaaba700347968"}], "type": "image", "id": "ebc5b93c6e8544dab542e1275235b904", "name": "glance"}, {"endpoints": [{"region_id": "RegionOne", "url": "http://172.16.2.72:9696", "region": "RegionOne", "interface": "admin", "id": "586c122ad2074349895e7987a622d7bd"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:9696", "region": "RegionTwo", "interface": "admin", "id": "58ca2bb600ed4914b892999738538c4c"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:9696", "region": "RegionTwo", "interface": "public", "id": "6e6d5d86be474d8099ae4601b25b4137"}, {"region_id": "RegionTwo", "url": "http://172.16.3.72:9696", "region": "RegionTwo", "interface": "internal", "id": "88ca57e4de734ce18bcf26e179294a36"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:9696", "region": "RegionOne", "interface": "public", "id": "d7b9ad90248e4d3e95cb51a67fd4e8f3"}, {"region_id": "RegionOne", "url": "http://172.16.2.72:9696", "region": "RegionOne", "interface": "internal", "id": "df754531d10046eb81d6abc8ae0cb516"}], "type": "network", "id": "ef52cd0a849a498791ddbc919c0c1f25", "name": "neutron"}], "user": {"password_expires_at": null, "domain": {"id": "default", "name": "Default"}, "id": "6fad19afd996469cb90ba18e063233f4", "name": "admin"}, "audit_ids": ["QMkfBlQ6RqCfTJbCE_8Mig"], "issued_at": "2021-08-20T08:24:33.000000Z"}}
```

Token nằm ở sau giá trị `X-Subject-Token`

Có 2 giá trị:

- "expires_at": "2021-08-20T09:24:33.000000Z": thời điểm hết hạn của Token

- "issued_at": "2021-08-20T08:24:33.000000Z": thời điểm Token được tạo

Ta thấy thời gian tồn tại mặc định của token là 1 tiếng (60 phút)

Export Token để sử dụng thuận tiện:

```
export OS_TOKEN=<giá_trị_token>
```

### 2.2. List user

**Command:**

```
[root@controller1 ~]# openstack user list
+----------------------------------+-----------+
| ID                               | Name      |
+----------------------------------+-----------+
| 0ef21a6be32e4ba4bffe11f28c75bf76 | anhtq     |
| 1654c825d7694024a4117e7725d7226f | neutron   |
| 442c50dd03f74541b1d6f641eb64b72a | demo      |
| 51f06e8ed4ce40b8b258fe6707bac74a | cinder    |
| 6fad19afd996469cb90ba18e063233f4 | admin     |
| 7c9d01f216b5487282c4b529437cd7c0 | placement |
| d2640c493ec34b06b73ff08960114356 | nova      |
| ff05bfa9f04f4106afecd37b7844228a | glance    |
+----------------------------------+-----------+
```

`curl` API
```
curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" http://localhost:5000/v3/users | python -mjson.tool
```

- `python -mjson.tool` : định dạng output ra là dạng json
- `-s` (--silent) : chế độ im lặng, sử dụng option này sẽ không hiển thị thông báo tiến trình hoặc thông báo lỗi

Kết quả:

```
[root@controller1 ~]# curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" http://localhost:5000/v3/users | python -mjson.tool
{
    "links": {
        "next": null,
        "previous": null,
        "self": "http://localhost:5000/v3/users"
    },
    "users": [
        {
            "domain_id": "default",
            "enabled": true,
            "id": "0ef21a6be32e4ba4bffe11f28c75bf76",
            "links": {
                "self": "http://localhost:5000/v3/users/0ef21a6be32e4ba4bffe11f28c75bf76"
            },
            "name": "anhtq",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "1654c825d7694024a4117e7725d7226f",
            "links": {
                "self": "http://localhost:5000/v3/users/1654c825d7694024a4117e7725d7226f"
            },
            "name": "neutron",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "442c50dd03f74541b1d6f641eb64b72a",
            "links": {
                "self": "http://localhost:5000/v3/users/442c50dd03f74541b1d6f641eb64b72a"
            },
            "name": "demo",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "51f06e8ed4ce40b8b258fe6707bac74a",
            "links": {
                "self": "http://localhost:5000/v3/users/51f06e8ed4ce40b8b258fe6707bac74a"
            },
            "name": "cinder",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "6fad19afd996469cb90ba18e063233f4",
            "links": {
                "self": "http://localhost:5000/v3/users/6fad19afd996469cb90ba18e063233f4"
            },
            "name": "admin",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "7c9d01f216b5487282c4b529437cd7c0",
            "links": {
                "self": "http://localhost:5000/v3/users/7c9d01f216b5487282c4b529437cd7c0"
            },
            "name": "placement",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "d2640c493ec34b06b73ff08960114356",
            "links": {
                "self": "http://localhost:5000/v3/users/d2640c493ec34b06b73ff08960114356"
            },
            "name": "nova",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "ff05bfa9f04f4106afecd37b7844228a",
            "links": {
                "self": "http://localhost:5000/v3/users/ff05bfa9f04f4106afecd37b7844228a"
            },
            "name": "glance",
            "options": {},
            "password_expires_at": null
        }
    ]
}
```

### 2.3. List project
Tương tự list user, thay thế URL

```
curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" \
http://localhost:5000/v3/projects | python -mjson.tool
```

### 2.4. List group

```
curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" \
http://localhost:5000/v3/groups | python -mjson.tool
```

### 2.5. List role

```
curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" \
http://localhost:5000/v3/roles | python -mjson.tool
```

### 2.6. List domain

```
curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" \
http://localhost:5000/v3/domains | python -mjson.tool
```

### 2.7. Tạo domain

```
curl -X POST -s -H "X-Auth-Token: $OS_TOKEN" -H "Content-Type: application/json" -d '{
  "domain": {
    "name": "domain_anhtq",
    "description": "Domain Anhtq"
  }
}' http://localhost:5000/v3/domains | python -mjson.tool
```

Kết quả:

```
{
    "domain": {
        "description": "Domain Anhtq",
        "enabled": true,
        "id": "594e5bcf30a64d808735662bf3f471c8",
        "links": {
            "self": "http://localhost:5000/v3/domains/594e5bcf30a64d808735662bf3f471c8"
        },
        "name": "domain_anhtq",
        "tags": []
    }
}
```

Kiểm tra domain vừa tạo:

```
[root@controller1 ~]# openstack domain show domain_anhtq
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Domain Anhtq                     |
| enabled     | True                             |
| id          | 594e5bcf30a64d808735662bf3f471c8 |
| name        | domain_anhtq                     |
| tags        | []                               |
+-------------+----------------------------------+
```

## Nguồn tham khảo

https://docs.openstack.org/api-ref/identity/v3/index.html