```
ceph version 14.2.8 (2d095e947a02261ce61424021bb43bd3022d35cb) nautilus (stable)
```
## Khi tạo bucket mới có hiện lỗi


```
500 - Internal Server Error
RGW REST API failed request with status code 416 '{"Code":"InvalidRange","BucketName":"1111111111","RequestId":"tx000000000000000000126-005e6f7c7e-fc18-default","HostId":"fc18-default-default"}'
```
## Check lỗi trong "/var/log/ceph/ceph-client.rgw.ceph1.log"  có xuất hiện:

```
rgw_init_ioctx ERROR: librados::Rados::pool_create returned (34) Numerical result out of range (this can be due to a pool or placement group misconfiguration, e.g. pg_num < pgp_num or mon_max_pg_per_osd exceeded)
```
## - Nguyên nhân: có thể do quá nhiều PGs trên OSD.
## - Giải quyết: điều chỉnh lại PGs, xoá bớt pool ko cần thiết

Links tham khảo:
https://ceph-users.ceph.narkive.com/zwPBOjFr/luminous-rgw-errors-at-start
