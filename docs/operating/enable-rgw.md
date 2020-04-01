Enabling the Object Gateway Management Frontend

To use the Object Gateway management functionality of the dashboard, you will need to provide the login credentials of a user with the system flag enabled.

If you do not have a user which shall be used for providing those credentials, you will also need to create one:

```sh
sudo radosgw-admin user create --uid=<user_id> --display-name=<display_name> --system
```
Take note of the keys access_key and secret_key in the output of this command.

The credentials of an existing user can also be obtained by using radosgw-admin:

```sh
sudo radosgw-admin user info --uid=<user_id>
```
Finally, provide the credentials to the dashboard:

```sh
sudo ceph dashboard set-rgw-api-access-key <access_key>
sudo ceph dashboard set-rgw-api-secret-key <secret_key>
```
If you are using a self-signed certificate in your Object Gateway setup, then you should disable certificate verification in the dashboard to avoid refused connections, e.g. caused by certificates signed by unknown CA or not matching the host name:

```sh
sudo ceph dashboard set-rgw-api-ssl-verify False
```
Disable/enable dashboard

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
