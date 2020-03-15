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



## Links tham khảo
https://www.virtualtothecore.com/adventures-with-ceph-storage-part-7-add-a-node-and-expand-the-cluster-storage/
https://medium.com/@george.shuklin/how-to-remove-osd-from-ceph-cluster-b4c37cc0ec87
