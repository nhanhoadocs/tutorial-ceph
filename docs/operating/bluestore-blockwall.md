# Sử dụng Cached trên SSD lưu BlockDB và WALL cho OSD 

- Ví dụ ta có 2 ổ là `sdb` dùng để lưu block db + service pools và `sdc` để lưu data

- Lấy key bootstrap OSD

`ceph auth get client.bootstrap-osd -o /var/lib/ceph/bootstrap-osd/ceph.keyring`

- Kiểm tra

`ceph-volume lvm list`

- Tạo volume group cho block data

`vgcreate ceph-block-0 /dev/sdc`

- Tạo logical volume cho block data

`lvcreate -l 100%FREE -n block-0 ceph-block-0`

- Tạo volume group cho block db.

`vgcreate ceph-db-0 /dev/sdb`

- Tạo logical volume để lưu block db

`lvcreate -L 40GB -n db-0 ceph-db-0`

- Tạo logical volume để lưu services pool

`lvcreate -L 40GB -n index-0 ceph-db-0`

- Tạo osd lưu data

`ceph-volume lvm create --bluestore --data ceph-block-0/block-0 --block.db ceph-db-0/db-0`

- Tạo osd lưu service pool

`ceph-volume lvm create --bluestore --data ceph-db-0/index-0`
