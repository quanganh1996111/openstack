# File log của Keystone

File cấu hình log của Keystone:

```
/etc/keystone/logging.conf
```

**3 kiểu log:**

- formatter_minimal:

```
format=%(message)s
```

- formatter_normal

```
format=(%(name)s): %(asctime)s %(levelname)s %(message)s
```

- formatter_debug

```
format=(%(name)s): %(asctime)s %(levelname)s %(module)s %(funcName)s %(message)s
```

File log nằm ở: `/var/log/keystone/keystone.log`

## 1. Khi đăng nhập vào OPS

### Đúng user-sai pass

```
2021-08-20 14:16:22.310 10920 WARNING keystone.common.wsgi [req-b1b053ba-bf2e-458f-a26a-fa80127b1588 6fad19afd996469cb90ba18e063233f4 - - default -] Authorization failed. The request you have made requires authentication. from 172.16.2.72: Unauthorized: The request you have made requires authentication.
```

`6fad19afd996469cb90ba18e063233f4` -> user đăng nhập sai. Kiểm tra user nào:

```
[root@controller1 ~]# openstack user list
+----------------------------------+-----------+
| ID                               | Name      |
+----------------------------------+-----------+
| 0ef21a6be32e4ba4bffe11f28c75bf76 | anhtq     |
| 1654c825d7694024a4117e7725d7226f | neutron   |
| 442c50dd03f74541b1d6f641eb64b72a | demo      |
| 51f06e8ed4ce40b8b258fe6707bac74a | cinder    |
| 6fad19afd996469cb90ba18e063233f4 | admin     | <= user admin
| 7c9d01f216b5487282c4b529437cd7c0 | placement |
| d2640c493ec34b06b73ff08960114356 | nova      |
| ff05bfa9f04f4106afecd37b7844228a | glance    |
+----------------------------------+-----------+
```

### Sai User

```
2021-08-20 14:22:20.382 10919 WARNING keystone.auth.plugins.core [req-769ddc09-7b90-4b23-b930-7ac61b4dbd91 - - - - -] Could not find user: saiuser.: UserNotFound: Could not find user: saiuser.
2021-08-20 14:22:20.383 10919 WARNING keystone.common.wsgi [req-769ddc09-7b90-4b23-b930-7ac61b4dbd91 - - - - -] Authorization failed. The request you have made requires authentication. from 172.16.2.72: Unauthorized: The request you have made requires authentication.
```

Log ghi nhận: `Could not find user: saiuser` => Không tìm thấy user nào tên là `saiuser`

## 2. Khởi tạo máy ảo

```
[req-90290515-2228-4277-8361-6961cdafb252 d2640c493ec34b06b73ff08960114356 b61de562adb142dbadccee43f87e6448 - default default] GET http://172.16.2.72:5000/v3/auth/tokens
```

**Trong đó:**

- `req-90290515-2228-4277-8361-6961cdafb252`: Mã request

- `d2640c493ec34b06b73ff08960114356`: ID User `nova` làm nhiệm vụ tạo VM.

- `b61de562adb142dbadccee43f87e6448`: ID Project

- `default default` : ID và tên domain

## 3. Rebuild, resize, suspend, start, ,create, delete VM

**Keystone không có log**

## 4. File log Keystone trong http

File log: Ta có thể show các file log Keystone trong http

```
[root@controller1 ~]# ls -alh /var/log/httpd/ | grep keystone
-rw-r--r--   1 root root 622K Aug 20 14:41 keystone_access.log
-rw-r--r--   1 root root    0 Aug 18 09:16 keystone.log
```