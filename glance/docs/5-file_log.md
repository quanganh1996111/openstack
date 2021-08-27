# File log của Glance

File log của Glance: `/var/log/glance/api.log`

```
2021-08-26 23:16:02.017 16091 INFO eventlet.wsgi.server [req-f3094553-b321-439e-aace-e4bdbe6c7d20 e06e2ee5649b44aa8271c8614aaa0344 c52a15d316c54dc4931e1c8d52686b98 - default default] 172.16.3.24 - - [26/Aug/2021 23:16:02] "GET /v2/schemas/image HTTP/1.1" 200 4383 1.244545
```

- `req-f3094553-b321-439e-aace-e4bdbe6c7d20`: Mã Request

- `e06e2ee5649b44aa8271c8614aaa0344`: ID User

- `c52a15d316c54dc4931e1c8d52686b98`: ID project

- `default default`: Domain

- `172.16.3.24`: Địa chỉ IP

- `"GET /v2/schemas/image HTTP/1.1" 200 4383 1.244545`: Thông tin method