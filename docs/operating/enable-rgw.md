## Bật Object Gateway Management(OGM). 

Để dùng được OGM chúng ta cần cung cấp thông tài khoản có cờ "system".

Cú pháp tạo user với cờ "system"

```sh
sudo radosgw-admin user create --uid=<user_id> --display-name=<display_name> --system
```
Lưu lại access_key và secret_key.

Command sử dụng trong trường hợp không nhớ access_key và secret_key của use:

```sh
sudo radosgw-admin user info --uid=<user_id>
```

Command gán quyền truy xuất dashboard cho user:

```sh
sudo ceph dashboard set-rgw-api-access-key <access_key>
sudo ceph dashboard set-rgw-api-secret-key <secret_key>
```

Nếu sử dụng chứng chỉ tự ký, có thể sẽ gặp một số lỗi về chứng chỉ, vậy chúng ta sẽ tắt xác nhận ssl của rgw: 

```sh
sudo ceph dashboard set-rgw-api-ssl-verify False
```
Tắt/bật lại dashboard

```sh
sudo ceph mgr module disable dashboard
sudo ceph mgr module enable dashboard
```
Khởi tạo node rgw trên cả 3 node ceph

```sh
cd ceph-deploy
ceph-deploy install --rgw ceph01 ceph02 ceph03
ceph-deploy rgw create ceph01 ceph02 ceph03 
```
Ceph Object Gateway được chạy trên Civetweb (được tích hợp sẵn trong ceph-radosgw daemon) bao gồm Apache và FastCGI. Civetweb của RGW chạy dưới port 7480
