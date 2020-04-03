## Cài đặt rgw trên các node
```sh
cd /ceph-deploy
ceph-deploy install --rgw ceph01 ceph02 ceph03
ceph-deploy rgw create ceph01 ceph02 ceph03 
```

## Tạo user (tuỳ chọn)
```sh
ceph dashboard ac-user-create <username> <password> administrator
```
## Lấy acesskey, secret key của user
```sh
radosgw-admin user info --uid=<username>
```

## Đặt quyền truy cập cho user 
```sh
#radosgw-admin user create --uid=<username> --display-name=<display_name> --system
ceph dashboard set-rgw-api-access-key <access_key>
ceph dashboard set-rgw-api-secret-key <secret_key>
```

Links tham khảo:
https://docs.ceph.com/docs/nautilus/mgr/dashboard/#enabling-the-object-gateway-management-frontend
https://www.gitmemory.com/issue/rook/rook/3026/529326017

