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