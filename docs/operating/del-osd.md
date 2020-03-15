## Kiểm tra trạng thái cluster

```sh 
ceph osd tree
```
## Thực hiện gỡ osd (ví dụ osd có id là 6, trên ceph4)
```sh
sudo ceph osd out osd.6
ssh ceph4 "sudo systemctl stop ceph-osd@6"
ssh ceph4 "sudo umount /var/lib/ceph/osd/ceph-6"
sudo ceph osd crush remove osd.6
sudo ceph auth del osd.6
sudo ceph osd rm osd.6
ceph-deploy purge ceph4
```



