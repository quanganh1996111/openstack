# Một số lệnh thường dùng với KeyStone

## 1. Khởi tạo biến môi trường

File này chứa thông tin credential (thông tin đăng nhập)

Tạo file khởi tạo biến môi trường:(tên file nên đặt theo user cho dễ nhớ)

Đứng từ thư mục `/root`:

```
[root@controller1 ~]# cat admin-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=013279227Anh
export OS_AUTH_URL=http://172.16.2.72:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

- Kiểm tra token:

```
openstack token issue
```

```
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2021-08-20T02:40:38+0000                                                                                                                                                                |
| id         | gAAAAABhHwgW5a2ncPCov6KjUWakROPX2NVB02n63KMpaC8rpHtUW2n0L9nPjbMFtdJgnMO_Cnc1TrEmiLnjmQGOntCsSTWUxVIOHnKgt_0U85GePR_5VnarvbjAjD0w7JT3k-7SRxGT3i5XLKH99Y9ZAZccFyiXJw8fxM_Pr6syQ4qUZ2bATE0 |
| project_id | a21e39dfc7f542cdb3d350a21165b5ca                                                                                                                                                        |
| user_id    | 6fad19afd996469cb90ba18e063233f4                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

## 2. Các lệnh làm việc với user, project, groups, roles, domain

### 2.1. Users

#### Liệt kê các Users

- List các user:

```
openstack user list
```

```
[root@controller1 ~]# openstack user list
+----------------------------------+-----------+
| ID                               | Name      |
+----------------------------------+-----------+
| 1654c825d7694024a4117e7725d7226f | neutron   |
| 442c50dd03f74541b1d6f641eb64b72a | demo      |
| 51f06e8ed4ce40b8b258fe6707bac74a | cinder    |
| 6fad19afd996469cb90ba18e063233f4 | admin     |
| 7c9d01f216b5487282c4b529437cd7c0 | placement |
| d2640c493ec34b06b73ff08960114356 | nova      |
| ff05bfa9f04f4106afecd37b7844228a | glance    |
+----------------------------------+-----------+
```

- List users trong 1 group::

```
openstack user list --group <group_name>
```

#### Tạo User

```
openstack user create <user_name> --password <password>
```

```
[root@controller1 ~]# openstack user create test --password 013279227AnhTQ
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 0ef21a6be32e4ba4bffe11f28c75bf76 |
| name                | test                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

#### Thay đổi thông tin User

Ta có thể cập nhật `name`, `email`, và `enable status` cho user

- Show thông tin User:

```
openstack user show <user>
```

- Disable / Enable User:

```
openstack user set <user> --disable
```

```
openstack user set <user> --enable
```

- Thay đổi tên và mô tả cho tài khoản người dùng:

```
openstack user set <user> --name <user-new>
```

Ví dụ:

```
[root@controller1 ~]# openstack user set test --name anhtq
[root@controller1 ~]# openstack user show anhtq
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 0ef21a6be32e4ba4bffe11f28c75bf76 |
| name                | anhtq                            |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

#### Xóa user

```
openstack user delete <user>
```

### 1.2. Project

#### List project

Liệt kê các project với lệnh sau:

```
openstack project list
```

```
+----------------------------------+---------+
| ID                               | Name    |
+----------------------------------+---------+
| a1067b80f045448cad1c973ef4b6ee76 | demo    |
| a21e39dfc7f542cdb3d350a21165b5ca | admin   |
| b61de562adb142dbadccee43f87e6448 | service |
+----------------------------------+---------+
```

#### Tạo project

```
openstack project create --description '<Description_Project>' <project_name> \
--domain <domain_name>
```

```
[root@controller1 ~]# openstack project create --description 'Test Project' test --domain default
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Test Project                     |
| domain_id   | default                          |
| enabled     | True                             |
| id          | d3ac0ba542b94e1c8853a61e9d36041d |
| is_domain   | False                            |
| name        | test                             |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+
```

#### Update project

Để cập nhật project bạn cần có ID hoặc tên của project

- Disable project:

```
openstack project set <project_ID || project_name> --disable
```

- Enable project:

```
openstack project set <project_ID || project_name> --enable
```

- Update tên cho project:

```
openstack project set <project_ID || project_name> --name <project_newname>
```

- Xác nhận các thông tin của project:

```
openstack project show <PROJECT_ID || project_name>
```

```
[root@controller1 ~]# openstack project show test
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Test Project                     |
| domain_id   | default                          |
| enabled     | False                            |
| id          | d3ac0ba542b94e1c8853a61e9d36041d |
| is_domain   | False                            |
| name        | test                             |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+
```

#### Delete project

```
openstack project delete <project_ID || project_name>
```

### 1.3. Group

#### Tạo group

```
openstack group create
    [--domain <domain>]
    [--description <description>]
    [--or-show]
    <group-name>
```

- Ví dụ:

```
[root@controller1 ~]# openstack group create --domain default \
> --description 'New Group AnhTQ' \
> --or-show \
> anhtq_group
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | New Group AnhTQ                  |
| domain_id   | default                          |
| id          | ce631567435f42baa33ee97578dcba1e |
| name        | anhtq_group                      |
+-------------+----------------------------------+
```

#### Liệt kê group

```
openstack group list
```

#### Thêm user vào group

```
openstack group add user
    [--group-domain <group-domain>]
    [--user-domain <user-domain>]
    <group>
    <user>
    [<user> ...]
```

- Ví dụ:

```
openstack group add user \
--group-domain default \
--user-domain default \
anhtq_group \
anhtq
```

Kiểm tra lại:

```
[root@controller1 ~]# openstack user list --group anhtq_group
+----------------------------------+-------+
| ID                               | Name  |
+----------------------------------+-------+
| 0ef21a6be32e4ba4bffe11f28c75bf76 | anhtq |
+----------------------------------+-------+
```

#### Kiểm tra user có nằm trong group không

```
openstack group contains user
    [--group-domain <group-domain>]
    [--user-domain <user-domain>]
    <group>
    <user>
```

- Ví dụ:

```
[root@controller1 ~]# openstack group contains user \
--group-domain default \
--user-domain default \
anhtq_group \
anhtq
anhtq in group anhtq_group
```

#### Remove user ra khỏi group

```
openstack group remove user
    [--group-domain <group-domain>]
    [--user-domain <user-domain>]
    <group>
    <user> [<user> ...]
```

- Ví dụ:

```
openstack group remove user \
--group-domain default \
--user-domain default \
anhtq_group \
anhtq
```

Kiểm tra lại:

```
openstack group contains user \
--group-domain default \
--user-domain default \
anhtq_group \
anhtq
anhtq not in group anhtq_group
```

#### Group set

Ta có thể đặt lại các thuộc tính của Group

```
openstack group set
    [--domain <domain>]
    [--name <name>]
    [--description <description>]
    <group>
```

#### Group show

Hiển thị thông tin chi tiết Group

```
openstack group show
    [--domain <domain>]
    <group>
```

### 2.4. Role

#### List avaiable roles

```
openstack role list
```

```[root@controller1 ~]# openstack role list
+----------------------------------+-------+
| ID                               | Name  |
+----------------------------------+-------+
| 3be3b4945c264f26b7e8936ea31dba10 | admin |
| 42bb756442d744d8a5e3319204e31ecf | user  |
+----------------------------------+-------+
```

#### Create a role

Để gán người dùng cho nhiều project, hay xác định vai role cho user với các project

```
openstack role create <role_name>
```

```
[root@controller1 ~]# openstack role create anhtq_role
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | None                             |
| id        | 0db064c731bd415ab70106889d2bddb1 |
| name      | anhtq_role                       |
+-----------+----------------------------------+
```

**Chú ý:** Nếu sử dụng Identity v3 thì cần có option `--domain` với một domain cụ thể

#### Assign a role

Để assign user cho project, ta cần gán role theo cặp user-project. Để làm điều đó, ta cần user ID, role và projectID

- List user và để ý userID mà ta muốn sử dụng

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

- List Role ID

```
[root@controller1 ~]# openstack role list
+----------------------------------+------------+
| ID                               | Name       |
+----------------------------------+------------+
| 0db064c731bd415ab70106889d2bddb1 | anhtq_role |
| 3be3b4945c264f26b7e8936ea31dba10 | admin      |
| 42bb756442d744d8a5e3319204e31ecf | user       |
+----------------------------------+------------+
```

- List project ID

```
[root@controller1 ~]# openstack project list
+----------------------------------+---------+
| ID                               | Name    |
+----------------------------------+---------+
| a1067b80f045448cad1c973ef4b6ee76 | demo    |
| a21e39dfc7f542cdb3d350a21165b5ca | admin   |
| b61de562adb142dbadccee43f87e6448 | service |
| ca52c2c6ff5e431f914d3ca99a50a4e6 | anhtq   |
+----------------------------------+---------+
```

- Assign role:

```
openstack role add --user <user> --project <project_name> <role_name>
```

Xác nhận lại :

```
[root@controller1 ~]# openstack role assignment list --user anhtq --project anhtq --names
+------------+---------------+-------+---------------+--------+-----------+
| Role       | User          | Group | Project       | Domain | Inherited |
+------------+---------------+-------+---------------+--------+-----------+
| anhtq_role | anhtq@Default |       | anhtq@Default |        | False     |
+------------+---------------+-------+---------------+--------+-----------+
```

#### Xem chi tiết role

```
openstack role show anhtq_role
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | None                             |
| id        | 0db064c731bd415ab70106889d2bddb1 |
| name      | anhtq_role                       |
+-----------+----------------------------------+
```

#### Remove role

```
openstack role remove --user <user> --project <project_name> <role_name>
```

Ví dụ:

```
openstack role remove --user anhtq --project anhtq anhtq_role
```

- Kiểm tra việc remove:

```
openstack role assignment list --user <user> --project <project_name|project_ID> --names
```

```
openstack role remove --user anhtq --project anhtq anhtq_role
```

## Nguồn tham khảo

https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/role-assignment.html

https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/group.html

https://docs.openstack.org/keystone/pike/admin/cli-manage-projects-users-and-roles.html