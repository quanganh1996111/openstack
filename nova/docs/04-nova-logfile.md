# File log của Nova

## /var/log/nova/nova-api.log

Log khi ta thực hiện các request đến API như tạo, xóa, tắt bật các VM

```
2021-08-29 10:38:00.800 1246 INFO nova.osapi_compute.wsgi.server [req-be9f9aaa-ab04-4dec-849e-f31555d18f89 - - - - -] (1246) wsgi starting up on http://172.16.3.24:8774
2021-08-29 10:38:00.870 1249 INFO nova.osapi_compute.wsgi.server [req-1db54630-a62c-4507-9eaf-ffee852760f8 - - - - -] (1249) wsgi starting up on http://172.16.3.24:8774
2021-08-29 10:38:00.877 1250 INFO nova.metadata.wsgi.server [req-aa16462a-8cfa-45c6-8a75-b25b8d3c9d15 - - - - -] (1250) wsgi starting up on http://172.16.3.24:8775
2021-08-29 10:38:00.954 1253 INFO nova.metadata.wsgi.server [req-b7a73702-8b9d-4870-a78d-eeafa9f04db7 - - - - -] (1253) wsgi starting up on http://172.16.3.24:8775
2021-08-29 10:38:00.959 1252 INFO nova.metadata.wsgi.server [req-5a6a8baa-b226-4936-a673-026bdeaa253a - - - - -] (1252) wsgi starting up on http://172.16.3.24:8775
2021-08-29 10:38:00.999 1247 INFO nova.osapi_compute.wsgi.server [req-f8906afc-8049-4257-9fdc-ccc7f1c37ad9 - - - - -] (1247) wsgi starting up on http://172.16.3.24:8774
2021-08-29 10:38:01.011 1248 INFO nova.osapi_compute.wsgi.server [req-0175d4d4-6d89-4686-9813-8ac632bd52d5 - - - - -] (1248) wsgi starting up on http://172.16.3.24:8774
```

## /var/log/nova/nova-conductor.log

```
2021-08-29 12:25:16.674 27499 WARNING oslo.service.loopingcall [-] Function 'nova.servicegroup.drivers.db.DbDriver._report_state' run outlasted interval by 1.31 sec
2021-08-29 12:25:20.205 27501 WARNING oslo.service.loopingcall [-] Function 'nova.servicegroup.drivers.db.DbDriver._report_state' run outlasted interval by 1.45 sec
2021-08-29 12:25:27.360 27499 WARNING oslo.service.loopingcall [-] Function 'nova.servicegroup.drivers.db.DbDriver._report_state' run outlasted interval by 0.27 sec
2021-08-29 12:25:41.615 27501 WARNING oslo.service.loopingcall [-] Function 'nova.servicegroup.drivers.db.DbDriver._report_state' run outlasted interval by 1.41 sec
2021-08-29 12:25:41.616 27499 WARNING oslo.service.loopingcall [-] Function 'nova.servicegroup.drivers.db.DbDriver._report_state' run outlasted interval by 4.26 sec
2021-08-29 12:25:41.618 27498 WARNING oslo.service.loopingcall [-] Function 'nova.servicegroup.drivers.db.DbDriver._report_state' run outlasted interval by 2.91 sec
2021-08-29 12:26:52.809 27498 WARNING oslo.service.loopingcall [-] Function 'nova.servicegroup.drivers.db.DbDriver._report_state' run outlasted interval by 1.17 sec
2021-08-29 12:26:52.819 27501 WARNING oslo.service.loopingcall [-] Function 'nova.servicegroup.drivers.db.DbDriver._report_state' run outlasted interval by 11.20 sec
2021-08-29 12:26:52.823 27500 WARNING oslo.service.loopingcall [-] Function 'nova.servicegroup.drivers.db.DbDriver._report_state' run outlasted interval by 24.00 sec
2021-08-29 13:45:32.659 27499 WARNING oslo.service.loopingcall [-] Function 'nova.servicegroup.drivers.db.DbDriver._report_state' run outlasted interval by 1.56 sec
2021-08-29 13:46:37.972 27499 WARNING oslo.service.loopingcall [-] Function 'nova.servicegroup.drivers.db.DbDriver._report_state' run outlasted interval by 5.17 sec
2021-08-29 13:46:37.973 27501 WARNING oslo.service.loopingcall [-] Function 'nova.servicegroup.drivers.db.DbDriver._report_state' run outlasted interval by 5.00 sec
2021-08-29 13:46:37.979 27498 WARNING oslo.service.loopingcall [-] Function 'nova.servicegroup.drivers.db.DbDriver._report_state' run outlasted interval by 4.84 sec
2021-08-29 13:46:37.981 27500 WARNING oslo.service.loopingcall [-] Function 'nova.servicegroup.drivers.db.DbDriver._report_state' run outlasted interval by 4.82 sec
```

## /var/log/nova/nova-manage.log

```
2021-08-29 10:13:47.735 22467 INFO migrate.versioning.api [-] 0 -> 1...
2021-08-29 10:13:47.790 22467 INFO migrate.versioning.api [-] done
2021-08-29 10:13:47.791 22467 INFO migrate.versioning.api [-] 1 -> 2...
2021-08-29 10:13:47.867 22467 INFO migrate.versioning.api [-] done
2021-08-29 10:13:47.868 22467 INFO migrate.versioning.api [-] 2 -> 3...
2021-08-29 10:13:47.928 22467 INFO migrate.versioning.api [-] done
2021-08-29 10:13:47.928 22467 INFO migrate.versioning.api [-] 3 -> 4...
2021-08-29 10:13:47.976 22467 INFO migrate.versioning.api [-] done
2021-08-29 10:13:47.977 22467 INFO migrate.versioning.api [-] 4 -> 5...
2021-08-29 10:13:48.070 22467 INFO migrate.versioning.api [-] done
2021-08-29 10:13:48.071 22467 INFO migrate.versioning.api [-] 5 -> 6...
2021-08-29 10:13:48.137 22467 INFO migrate.versioning.api [-] done
2021-08-29 10:13:48.138 22467 INFO migrate.versioning.api [-] 6 -> 7...
2021-08-29 10:13:48.197 22467 INFO migrate.versioning.api [-] done
2021-08-29 10:13:48.198 22467 INFO migrate.versioning.api [-] 7 -> 8...
2021-08-29 10:13:48.210 22467 INFO migrate.versioning.api [-] done
2021-08-29 10:13:48.211 22467 INFO migrate.versioning.api [-] 8 -> 9...
2021-08-29 10:13:48.224 22467 INFO migrate.versioning.api [-] done
2021-08-29 10:13:48.224 22467 INFO migrate.versioning.api [-] 9 -> 10...
2021-08-29 10:13:48.236 22467 INFO migrate.versioning.api [-] done
2021-08-29 10:13:48.237 22467 INFO migrate.versioning.api [-] 10 -> 11...
```

## /var/log/nova/nova-scheduler.log

```
2021-08-29 15:31:44.356 27489 INFO nova.scheduler.host_manager [req-7081d30e-13b0-40aa-9422-20b162b59739 38223d22cc044cf79855c81db9e2190e d6cd981091e2464dae9ba81b4fd32651 - default default] Host filter forcing available hosts to compute02
2021-08-29 15:31:58.468 27489 INFO nova.scheduler.host_manager [req-2587470b-909d-497f-97f9-95b0f2a94bef - - - - -] The instance sync for host 'compute02' did not match. Re-created its InstanceList.
2021-08-29 15:34:02.826 27489 INFO nova.scheduler.host_manager [req-63ee27b3-8981-42b1-9db0-680ff6ee0d5c - - - - -] The instance sync for host 'compute02' did not match. Re-created its InstanceList.
2021-08-29 15:37:06.333 27489 INFO nova.scheduler.host_manager [req-011f12da-0839-4140-ab01-394ae941666b 38223d22cc044cf79855c81db9e2190e d6cd981091e2464dae9ba81b4fd32651 - default default] Host filter forcing available hosts to compute02
2021-08-29 15:48:18.313 27489 INFO nova.scheduler.host_manager [req-75b5b46f-657f-4eaa-9690-e03ce2968d54 38223d22cc044cf79855c81db9e2190e d6cd981091e2464dae9ba81b4fd32651 - default default] Host filter forcing available hosts to compute02
2021-08-29 15:54:21.455 27489 INFO nova.scheduler.host_manager [req-022e20e5-b659-44fc-b1b1-cdb634fb1c10 38223d22cc044cf79855c81db9e2190e d6cd981091e2464dae9ba81b4fd32651 - default default] Host filter forcing available hosts to compute02
```