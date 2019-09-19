# Môi trường giả định Node hỏng vật lý 

Gắn các OSD từ node hỏng sang node mới và thực hiện 
```
ceph-volume lvm activate --all
```

Điều kiện 
- Có node vật lý mới thay thế 
- IP planning giống như node vừa hỏng 
- Hostname giống như node vừa hỏng 
- Cài đặt ceph version giống như node vừa hỏng (Chưa kiểm chứng vụ khác version)
- Copy ceph.conf của cụm sang node mới 
- Tiến hành active các lvm trên các ổ

Thông tin metadata của OSD cần phải gần khớp với node hỏng 
```sh 
{
    "id": 4,
    "arch": "x86_64",
    "back_addr": "192.168.83.79:6802/12775",
    "back_iface": "eth2",
    "bluefs": "1",
    "bluefs_db_access_mode": "blk",
    "bluefs_db_block_size": "4096",
    "bluefs_db_dev": "252:1",
    "bluefs_db_dev_node": "dm-1",
    "bluefs_db_driver": "KernelDevice",
    "bluefs_db_model": "",
    "bluefs_db_partition_path": "/dev/dm-1",
    "bluefs_db_rotational": "1",
    "bluefs_db_size": "21470642176",
    "bluefs_db_type": "hdd",
    "bluefs_single_shared_device": "1",
    "bluestore_bdev_access_mode": "blk",
    "bluestore_bdev_block_size": "4096",
    "bluestore_bdev_dev": "252:1",
    "bluestore_bdev_dev_node": "dm-1",
    "bluestore_bdev_driver": "KernelDevice",
    "bluestore_bdev_model": "",
    "bluestore_bdev_partition_path": "/dev/dm-1",
    "bluestore_bdev_rotational": "1",
    "bluestore_bdev_size": "21470642176",
    "bluestore_bdev_type": "hdd",
    "ceph_version": "ceph version 12.2.12 (1436006594665279fe734b4c15d7e08c13ebd777) luminous (stable)",
    "cpu": "Intel Xeon E312xx (Sandy Bridge, IBRS update)",
    "default_device_class": "hdd",
    "distro": "centos",
    "distro_description": "CentOS Linux 7 (Core)",
    "distro_version": "7",
    "front_addr": "192.168.82.79:6802/12775",
    "front_iface": "eth1",
    "hb_back_addr": "192.168.83.79:6803/12775",
    "hb_front_addr": "192.168.82.79:6803/12775",
    "hostname": "ceph3-test",
    "journal_rotational": "1",
    "kernel_description": "#1 SMP Wed Jun 5 14:26:44 UTC 2019",
    "kernel_version": "3.10.0-957.21.2.el7.x86_64",
    "mem_swap_kb": "2097148",
    "mem_total_kb": "1882080",
    "os": "Linux",
    "osd_data": "/var/lib/ceph/osd/ceph-4",
    "osd_objectstore": "bluestore",
    "rotational": "1"
}
```

Đã được kiểm chứng : 2019-06-15 CanhDX Nhân Hòa Cloud Team 

Môi trường 
- CentOS 7 1810 cùng version 
- Ceph 12.2.12 cùng version 
- Interface name giống nhau 
- IP planning giống nhau 
- .. 