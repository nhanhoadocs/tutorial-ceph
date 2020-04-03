# Storage cơ bản 

[Các note ghi chép liên quan đến Storage](https://github.com/uncelvel/storage)

# Lý thuyết Ceph Storage

[Ceph - Overview](docs/knowledge_base/ceph-overview.md)

# Các services, thành phần của Ceph

- [Ceph RADOS](docs/knowledge_base/ceph-rados.md)
- [CRUSH](docs/knowledge_base/crush.md)
- [Ceph Storage Backend](docs/knowledge_base/bluestore_vs_filestore.md)
- [Ceph MON - Monitor (ceph-mon)](docs/knowledge_base/ceph-mon.md)
- [Ceph OSD - Object Storage Device (ceph-osd)](docs/knowledge_base/ceph-osd.md)
- [Ceph RBD - RADOS Block Device](docs/knowledge_base/ceph-rbd.md)
- [Ceph MDS - Metadata Server (ceph-mds)](docs/knowledge_base/ceph-mds.md)
- [Ceph RADOSGW - Object Gateway(ceph-radosgw)](docs/knowledge_base/ceph-radosgw.md)
- [Ceph MGR - Manager (ceph-mgr)](docs/knowledge_base/ceph-mgr.md)
- [Thuật toán PAXOS](docs/knowledge_base/paxos.md)
- [Cơ chế xác thực của Ceph](docs/knowledge_base/ceph-authen.md)
- [Các flag của Cluster Ceph](docs/knowledge_base/ceph-flag.md)
- [Các trạng thái của PG](docs/knowledge_base/ceph-pg-status.md)

# Tài liệu cài đặt

[Cài đặt CephAIO bản Luminous sử dụng scripts](https://github.com/uncelvel/script-ceph-lumi-aio)

[Cài đặt CephAIO bản Luminous manual-cephuser](docs/setup/ceph-luminous-aio.md)

[Cài đặt Ceph bản Luminous](docs/setup/ceph-luminous.md)

[Cài đặt Ceph bản Mimic](docs/setup/ceph-mimic.md)

[Cài đặt Ceph bản Nautilus](docs/setup/ceph-nautilus.md)

[Cài đặt Ceph bản Octopus](docs/setup/ceph-octopus.md)

[Cài đặt Ceph-RadosGW HA bản Nautilus](docs/setup/ceph-radosgw.md)

# Tài liệu tích hợp

[Tích hợp Linux Client sử dụng RBD Ceph](docs/operating/ceph-vs-client-linux.md)

[Sử dụng CephFS (File Storage) cơ bản](docs/operating/ceph-vs-client-linux.md)

[Sử dụng RBD (Block Storage) cơ bản]()

[Sử dụng RGW (Object Storage) cơ bản]()

[Tích hợp Ceph với OpenStack](docs/operating/ceph-vs-openstack.md)

# Tài liệu vận hành


## Thêm osd

[Thêm osd](docs/operating/add-osd.md)

## Cập nhật osd

## Xoá osd

[Xoá osd](docs/operating/del-osd.md)

## Bật rgw

[Bật rgw](docs/operating/enable-rgw.md)

## Tắt/bật cluster

[Tắt/bật cluster](docs/operating/off-on-cluster.md)

## CheatSheet thao tác 

[Ceph Cheat sheet](docs/operating/ceph-cheat-sheet.md)

## Module của MGR

[Ceph Module Balancer](docs/operating/ceph-module-balancer.md)

# Benchmark & Troubleshooting

- [Lỗi không tạo bucket](docs/operating/bucket-err.md)

- [Node Ceph hỏng](docs/operating/ceph-hardware-crash.md)

- [Note case vận hành](docs/operating/note.md)
