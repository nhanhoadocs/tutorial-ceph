![](https://i.imgur.com/oNi9e15.png) 
---

# Các lệnh cơ bản

## ceph-deploy

- Install Ceph trên Client 
```sh 
ceph-deploy install {client}
```

- Khởi tạo cụm 
```sh 
ceph-deploy mon create-initial
```

- Copy key admin và config 
```sh 
ceph-deploy --overwrite-conf admin {host}
```

- Tạo mới OSD 
```sh 
# Zapdisk
ceph-deploy disk zap {host} /dev/{disk}
# Create OSD (BlueStore)
ceph-deploy osd create --data /dev/{disk} {host}
```

- Tạo mới Node Mon
```sh 
ceph-deploy new {host}
```

- Xóa Node Mon
```sh 
# Remove trong config và restart service mon
```

- Tạo mới Node MGR
```sh 
ceph-deploy mgr create {host}
```

- Xóa node MGR
```sh 
# 
```

- Push config mới qua các node client 
```sh 
ceph-deploy --overwrite-conf config push {host}
```

- Tạo mới node RGW
```sh 
ceph-deploy rgw create {host}
```

- Push key qua client 

- Xóa node 
```sh 
# Xóa data trong `/var/lib/ceph`
ceph-deploy purgedata {host} [{host2} {host3}]

# Xóa data và remove package Ceph
ceph-deploy purge {host} [{host2} {host3}]
```


## Restart Service Ceph

- Mon 
```sh 
systemctl restart ceph-mon@$(hostname)
```

- OSD
```sh 
systemctl restart ceph-osd@{osd-id}
```

- MDS
```sh 
systemctl restart ceph-mds@$(hostname)
```

- RGW
```sh 
systemctl status ceph-radosgw@rgw.$(hostname).service
```

- MGR
```sh 
systemctl restart ceph-mgr@$(hostname)
```

## Kiểm tra trạng thái hệ thống 

- Hiển thị trạng thái cụm Ceph
```sh 
ceph health
```

- Hiển thị chi tiết trạng thái Warning, Error
```sh 
ceph health detail
```

- Hiển thị chi tiết trạng thái cụm Ceph 
```sh 
ceph -s
```

- Hiển thị trạng thái cụm Ceph theo giờ gian thực 
```sh 
ceph -w
```

- Kiểm tra trạng thái sử dụng disk của mỗi pool
```sh 
ceph df
```

- Kiểm tra trạng thái sử dụng disk của mỗi pool theo Object
```sh 
rados df
```

## Các lệnh thao tác với MGR

- Kiểm tra thông tin các module của MGR
```sh 
[root@ceph1 ~]# ceph mgr dump
{
    "epoch": 20171,
    "active_gid": 94098,
    "active_name": "ceph1",
    "active_addr": "10.10.10.33:6806/1561",
    "available": true,
    "standbys": [],
    "modules": [
        "balancer",
        "restful",
        "status"
    ],
    "available_modules": [
        "balancer",
        "dashboard",
        "influx",
        "localpool",
        "prometheus",
        "restful",
        "selftest",
        "status",
        "zabbix"
    ],
    "services": {}
}
[root@ceph1 ~]#
```

- Enable các module MGR (zabbix, dashboard,...)
```sh 
ceph mgr module enable {module}
```

## Các lệnh thao tác với OSD 

- Kiểm tra OSD được create từ BlockDevice lvm nào 
```sh 
ceph-volume lvm list
```
    > Output 
    ```sh 
    ====== osd.1 =======

    [block]    /dev/ceph-3622c3ad-e4e5-4bdb-9399-1934becdae8f/osd-block-7e929d2e-5026-4590-8abd-c596d7e3e2e0

        type                      block
        osd id                    1
        cluster fsid              7d3f2102-face-4012-a616-372615f2f54f
        cluster name              ceph
        osd fsid                  7e929d2e-5026-4590-8abd-c596d7e3e2e0
        encrypted                 0
        cephx lockbox secret      
        block uuid                BYR5va-JZrp-qfWV-rngf-iazj-CEdN-A3SShX
        block device              /dev/ceph-3622c3ad-e4e5-4bdb-9399-1934becdae8f/osd-block-7e929d2e-5026-4590-8abd-c596d7e3e2e0
        vdo                       0
        crush device class        None
        devices                   /dev/sdc
    ```

- Hiển thị trạng thái các OSD trong cụm 
```sh 
[root@ceph1 ~]# ceph osd stat
9 osds: 9 up, 9 in
```

- Hiển thị tình trạng used, r/w, state của các osd 
```sh 
[root@ceph1 ~]# ceph osd status
+----+-------+-------+-------+--------+---------+--------+---------+-----------+
| id |  host |  used | avail | wr ops | wr data | rd ops | rd data |   state   |
+----+-------+-------+-------+--------+---------+--------+---------+-----------+
| 0  | ceph1 | 3896M |  138G |    0   |     0   |    0   |     0   | exists,up |
| 1  | ceph1 | 3511M |  146G |    0   |     0   |    0   |     0   | exists,up |
| 2  | ceph1 | 3158M |  146G |    0   |     0   |    0   |     0   | exists,up |
| 3  | ceph2 | 3750M |  146G |    0   |     0   |    0   |     0   | exists,up |
| 4  | ceph2 | 3444M |  146G |    0   |     0   |    0   |     0   | exists,up |
| 5  | ceph2 | 4023M |  146G |    0   |     0   |    0   |     0   | exists,up |
| 6  | ceph3 | 3662M |  146G |    0   |     0   |    0   |     0   | exists,up |
| 7  | ceph3 | 3741M |  146G |    0   |     0   |    0   |     0   | exists,up |
| 8  | ceph3 | 3244M |  146G |    0   |  2457   |    0   |     0   | exists,up |
+----+-------+-------+-------+--------+---------+--------+---------+-----------+
``` 

- Hiển thị Crushmap OSD 
```sh 
ceph osd tree
ceph osd crush tree
ceph osd crush tree --show-shadow
```

- Kiểm tra chi tiết location của 1 OSD 
```sh 
ceph osd find {osd-id}
```

- Kiểm tra chi tiết metadata của 1 OSD
```sh 
ceph osd metadata {osd-id}
```

- Benchmark osd 
```sh 
ceph tell osd.{osd-id} bench
```

- Hiển thị trạng thái sử dụng của các OSD 
```sh 
ceph osd df 
ceph osd df tree
```

- Hiển thị latency Aplly, Commit data trên các OSD 
```sh 
ceph osd perf
```

- Xóa 1 osd ra khỏi cụm Ceph (Thực hiện trên host của OSD đó)
```sh 
i={osd-id}
ceph osd out osd.$i
ceph osd down osd.$i
systemctl stop ceph-osd@$i
ceph osd crush rm osd.$i
ceph osd rm osd.$i
ceph auth del osd.$i
```

## Các lệnh thao tác trên pool

- Create 1 pool 
```sh 
ceph osd pool create {pool-name} {pg-num} [{pgp-num}] [replicated] \
     [crush-ruleset-name] [expected-num-objects]
```

- Set Application cho pool
```sh 
osd pool application enable {pool-name} {application}
```

- Hiển thị toàn bộ tham số của các pool 
```sh 
ceph osd pool ls detail
```

- Hiện thị tham số của 1 pool
```sh 
ceph osd pool get {pool-name} all
```

- Điều chỉnh lại giá trị của pool
```sh 
ceph osd pool set {pool-name} {key} {value}
```

- Xóa pool 
```sh
ceph osd pool delete {pool-name} {pool-name} --yes-i-really-really-mean-it
```

## Các lệnh thao tác xác thực trên Ceph

- Hiển thị toàn bộ các key authen của cụm Ceph
```sh 
ceph auth list
```

- Create hoặc get key 
```sh 
ceph auth get-or-create {key-name} mon {permission} osd {permission} mds {permission} > {key-name}.keyring
```

- Cập nhật permission key đã có 
```sh 
ceph auth caps {key-name} mon {permission} osd {permission} mds {permission}
```

- Xóa key
```sh 
ceph auth delete {key-name}
```

## Các lệnh thao tác đối với RBD

- Hiển thị các images trong pool
```sh 
rbd ls {pool-name}
```

- Create 1 images 
```sh 
rbd create {pool-name}/{images} --size {size}G
```

- Hiển thị chi tiết images 
```sh 
rbd info {pool-name}/{images}
```

- Hiển thị dung lượng thực tế của images
```sh 
rbd diff {pool-name}/{images} | awk '{SUM += $2} END {print SUM/1024/1024 " MB"}'
```

- Điều chỉnh dung lượng images 
```sh 
rbd resize {pool-name}/{images} --size {size}G
```

- Hiển thị images đang được mapped (Trên Client)
```sh 
rbd showmapped
```

- Xóa images
```sh 
rbd rm {pool-name}/{images}
```

- Create snapshot
```sh 
rbd snap create {pool-name}/{images}@{snap-name}
```

- Protect bản snapshot 
```sh 
rbd snap protect {pool-name}/{images}@{snap-name}
```

- Kiểm tra tất cả các bản snapshot của 1 volume 
```sh 
rbs snap ls {pool-name}/{images}
```

- Roolback snapshot 
```sh 
rbd snap rollback {pool-name}/{images}@{snap-name}
```

- Clone snapshot thành 1 images mới 
```sh 
rbd clone {pool-name}/{images}@{snap-name} {pool-name}/{child-images}
```

- Kiểm tra các images được clone từ snapshot
```sh 
rbd children {pool-name}/{images}@{snap-name}
```

- Tách hẳn images mới ra khỏi images parent 
```sh 
rbd flatten {pool-name}/{child-images}
```

- Unprotect bản snapshot 
```sh 
rbd snap unprotect {pool-name}/{images}@{snap-name}
```

- Xóa 1 bản snapshot 
```sh 
rbd snap rm {pool-name}/{images}@{snap-name}
```

- Xóa toàn bộ snapshot của 1 volume 
```sh 
rbd snap purge {pool-name}/{images}
```

- Export volume 
```sh 
rbd export --rbd-concurrent-management-ops 20 --pool={pool-name} {images} {images}.img
```

## Các lệnh thao tác đối với Object

- Show toàn bộ pool name 
```sh 
rados lspools
```

- Show toàn bộ Object trên cụm 
```sh 
rados -p {pool-name} ls
```

- Upload Object lên cụm Ceph
```sh 
rados -p {pool-name} put {object-file}
```

- Download Object từ cụm Ceph 
```sh 
rados -p {pool-name} get {object-file}
```

- Xóa 1 Object cụm Ceph 
```sh 
rados -p {pool-name} rm {object-file}
```

- Kiểm tra các client kết nối đến Object 
```sh 
rados -p {pool-name} listwatchers {object-file}
```

- Benchmark Object bằng rados bench 
```sh 
rados -p {pool-name} listwatchers {object-file}
```

## Các lệnh thao tác với CephFS

- Create pool cho cephfs 
```sh 
osd pool create <poolname> <int[0-]> {<int[0-]>} {replicated|erasure} {<erasure_code_profile>} {<rule>}   create pool
{<int>}   
ceph osd pool create cephfs_data 128
ceph osd pool create cephfs_metadata 128
```

- Khởi tạo cephfs 
```sh 
ceph fs new cephfs cephfs_metadata cephfs_data
```

- Kiểm tra các cephfs trên cụm
```sh 
ceph fs ls
ceph mds stat
```

- Enable nhiều cephfs trên cụm 
```sh 
ceph fs flag set enable_multiple true --yes-i-really-mean-it
```

## Thao tác trên PG

- Dump toàn bộ thông tin PG trên cụm 
```sh 
ceph pg dump [--format <format>]
```

- Dump thông tin cụ thể của PG_stuck
```sh 
ceph pg dump_stuck inactive|unclean|state|undersized|degraded
```

- Query thông tin của 1 PG
```sh 
ceph pg {pg-id} query
```

- List các pg missing 
```sh 
ceph pg {pg-id} list-missing
```


ceph pg scrub {pg-id}
ceph deep-scrub {pg-id}

- Chủ động khôi phục pg gặp sự cố 
```sh 
ceph pg repair {pg-id}
```

# Các lệnh nâng cao 

## Thao tác với flag của Ceph

Thao tác apply cho toàn Cluster
```
ceph osd set <flag>
ceph osd unset <flag>
```

- full - Set flag full chúng ta sẽ không thể ghi dữ liệu mới vào được cluster
- pauserd, pausewr - Tạm dừng thao tác read hoặc write trên cụm 
- noup - Khi set flag sẽ tạm ngưng thông báo khi có OSD up trong hệ thống
- nodown - Khi set flag sẽ tạm ngưng thông báo khi có OSD down trong hệ thống
- noin - Đối với các OSD đã down trước đó thì việc set flag này sẽ không tự động set OSD in và không cho phép OSD join vào CRUSHMAP khi OSD này start
- noout - Khi OSD down hệ thống sẽ không tự động set OSD out ra khỏi cluster (Thường là 10m) cho đến khi chúng ta gỡ flag
- nobackfill, norecover, norebalance - Suppend thao tác recovery và data rebalancing
- noscrub, nodeep_scrub - Disable scrubbing
- notieragent - Suppend cache tiering đang hoạt động


Thao tác set chỉ định OSD
```sh
ceph osd add-<flag> <osd-id>
ceph osd rm-<flag> <osd-id>
```

Đối với OSD chỉ định chỉ apply được các tham số 

- noup
- nodown
- noin
- noout

- Thực hiện add thêm ổ đối với cụm Ceph đang chạy 
```sh 
ceph osd reweight osd.{osd-id} {weight1} 
ceph osd crush reweight osd.{osd-id} {weight2}
```
Trong đó: 
    - {weight1}: Recommend là % sử dụng của cụm tương ứng với %use mà chúng ta muốn đẩy vào ổ mới 
    - {weight2}: Là dung lượng thực tế của ổ tính theo TiB, Ceph sẽ dựa trên tham số này để định lượng data đẩy vào OSD sau này 

## Show config ceph
- Show config trong file ceph.conf của Ceph 
```sh 
ceph-conf --show-conf
```

- Show toàn bộ config của Ceph 
```sh 
ceph-conf --show-config
```

- Kiểm tra dung lượng của các images hiện có
```sh 
for i in $(rbd ls {pool-name}); 
do 
echo {pool-name}/$i 
Size: $(rbd diff {pool-name}/$i | awk '{SUM += $2} END {print SUM/1024/1024 " MB"}'); 
done
```

##  Thao tác với CRUSHMAP

Các thao tác với crushmap file 

- Get Crushmap
```sh 
ceph osd getcrushmap -o crushmap
```

- Decompile crushmap file 
```sh 
crushtool -d crushmap -o crushmap.decom
```

- Sau khi chỉnh sửa crushmap tiến hành Compile lại 
```sh 
crushtool -c crushmap.decom -o crushmap.new
```

- Test apply crushmap mới cho cụm 
```sh 
crushtool --test -i crushmap.new --show-choose-tries --rule 2 --num-rep=2
```

- Apply crushmap cho cụm 
```sh 
ceph osd setcrushmap -i crushmap.new
```

Các thao tác với Crushmap trên câu lệnh 

- Set lại class của 1 OSD
```sh 
# Remove class cũ
ceph osd crush rm-device-class osd.{osd-id}
# Add lại class mới cho osd
ceph osd crush set-device-class ssd osd.{osd-id}
```

- List danh sách class của osd
```sh 
ceph osd crush class ls
```

- List danh sách osd theo class 
```sh 
ceph osd crush class ls-osd {class}
```

- Kiểm tra rules hiện có của cụm 
```sh 
ceph osd crush rule ls
```

- Kiểm tra chi tiết của 1 rule
```sh 
ceph osd crush rule dump
```

- Create rule mới 
```sg 
ceph osd crush rule create-replicated <rule-name> <root> <failure-domain> <class>
```

## EC pool
- Create rule cho ensure code 
```sh 
ceph osd erasure-code-profile set {profileEC} k=4 m=2 crush-device-class=ssd crush-failure-domain=host
```

- Create ecpool
```sh 
ceph osd pool create {ecpool-name} {pg_size} {pgp_size} erasure {profileEC}
```

- Set ecpool cho phép lưu trữ RBD
```sh 
ceph osd pool set {ecpool-name} allow_ec_overwrites true
ceph osd pool application enable {ecpool-name} rbd
```

- Create images 
```sg 
rbd create {ecpool-name}/{images} --size {size}
```

## Tùy chỉnh OSD trên Crushmap 

- Tùy chỉnh cụ thể osd trên Crushmap 
```sh 
ceph osd crush set osd.0 1.0 root=default datacenter=dc1 room=room1 row=foo rack=bar host=foo-bar-1
```
> VD trên set osd.0 có weight =1 và nằm dưới root/dc1/roomm1/foo/bar/foor-bar-1

- Xóa 1 bucket
```sh 
ceph osd crush rm {bucket-name}
```

## Ví dụ tùy chỉnh 

```sh
# Add host node4
ceph osd crush add-bucket node4 host
# Move node4 to root default
ceph osd crush move node4 root=default
# Add disktype node4hdd, node4ssd
ceph osd crush add-bucket node4hdd disktype
ceph osd crush add-bucket node4ssd disktype
# Move disktype node4hdd, node4ssd to host node4
ceph osd crush move node4ssd host=node4
ceph osd crush move node4hdd host=node4
# Add osd to disktype
ceph osd crush set osd.11 0.00999 disktype=node4ssd
ceph osd crush set osd.12 0.00999 disktype=node4hdd
```


## Tổng hợp thông tin PG của cụm
```sh 
ceph pg dump | awk '
BEGIN { IGNORECASE = 1 }
/^PG_STAT/ { col=1; while($col!="UP") {col++}; col++ }
/^[0-9a-f]+\.[0-9a-f]+/ { match($0,/^[0-9a-f]+/); pool=substr($0, RSTART, RLENGTH); poollist[pool]=0;
up=$col; i=0; RSTART=0; RLENGTH=0; delete osds; while(match(up,/[0-9]+/)>0) { osds[++i]=substr(up,RSTART,RLENGTH); up = substr(up, RSTART+RLENGTH) }
for(i in osds) {array[osds[i],pool]++; osdlist[osds[i]];}
}
END {
printf("\n");
printf("pool :\t"); for (i in poollist) printf("%s\t",i); printf("| SUM \n");
for (i in poollist) printf("--------"); printf("----------------\n");
for (i in osdlist) { printf("osd.%i\t", i); sum=0;
for (j in poollist) { printf("%i\t", array[i,j]); sum+=array[i,j]; sumpool[j]+=array[i,j] }; printf("| %i\n",sum) }
for (i in poollist) printf("--------"); printf("----------------\n");
printf("SUM :\t"); for (i in poollist) printf("%s\t",sumpool[i]); printf("|\n");
}'
```

## Lấy thông tin sử dụng thực tế và đã cấp của pool
```sh
pool="volumes"
echo > size.txt 
for vol in $(rbd ls $pool)
do
real=$(rbd diff $pool/$vol | awk '{SUM += $2} END {print SUM/1024/1024 }')
capa=$(rbd info $pool/$vol | grep size | awk '{print $2}' | cut -d 'G' -f1)
echo "$pool/$vol $real $capa" >> size.txt
done
sum_real=$(cat size.txt| awk '{SUM += $2} END {print SUM/1024} MB')
sum_capa=$(cat size.txt| awk '{SUM += $3} END {print SUM} GB')
echo -e "SUM_REAL:$sum_real GB\nSUM_CAPACITY:$sum_capa GB"
```

## Map Image RBD trên Client 

Map rbd như 1 Block Device
```sh 
# Trên client cài đặt cephclient
yum install ceph-common -y 

# Download ceph.conf và key về /etc/ceph/
scp root@cephnode:/etc/ceph/ceph.conf /etc/ceph/
scp root@cephnode:/etc/ceph/{key-name}.keyring /etc/ceph/

# Add config vào rbdmap 
echo "{pool-name}/{images}            id=admin,keyring=/etc/ceph/ceph.client.admin.keyring" >> /etc/ceph/rbdmap

# Kiểm tra 
sudo modprobe rbd
rbd feature disable {pool-name}/{images}  exclusive-lock object-map fast-diff deep-flatten
systemctl start rbdmap && systemctl enable rbdmap

# Bổ sung fstab 
echo "UUID=bfdf0e00-1d73-4bd9-a43e-32c408dbfdc9 /data ext4 noauto 0 0" >> /etc/fstab

# Nếu là sử dụng lvm thì bổ sung vào lvm.conf
types = [ "rbd", 1024 ]
```

## Map cephfs trên Client

Map cephfs như 1 NFS folder
```sh 
sudo mount -t ceph 192.168.0.1:6789:/ /mnt/mycephfs -o name=admin,secretfile=/etc/ceph/admin.secret

sudo mkdir /home/usernname/cephfs
sudo ceph-fuse -m 192.168.0.1:6789 /home/username/cephfs

# Cấu hình trên fstab
10.10.10.10:6789:/     /mnt/ceph    ceph    name=admin,secretfile=/etc/ceph/secret.key,noatime,_netdev    0       2
```

## Force unmount folder 

```sh 
rbd unmap -o force $DEV
```

# Cấu hình ceph.conf tham khảo
```sh 
[global]
# Requirement Config
fsid = 7d3f2102-face-4012-a616-372615f2f54f
mon_initial_members = ceph1
mon_host = 10.10.10.33
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx

# Network
public_network = 172.16.4.0/24
cluster_network = 10.0.0.0/24

# Debug config
debug_lockdep = 0/0
debug_context = 0/0
debug_crush = 0/0
debug_mds = 0/0
debug_mds_balancer = 0/0
debug_mds_locker = 0/0
debug_mds_log = 0/0
debug_mds_log_expire = 0/0
debug_mds_migrator = 0/0
debug_buffer = 0/0
debug_timer = 0/0
debug_filer = 0/0
debug_objecter = 0/0
debug_rados = 0/0
debug_rbd = 0/0
debug_journaler = 0/0
debug_objectcacher = 0/0
debug_client = 0/0
debug_osd = 0/0
debug_optracker = 0/0
debug_objclass = 0/0
debug_filestore = 0/0
debug_journal = 0/0
debug_ms = 0/0
debug_mon = 0/0
debug_monc = 0/0
debug_paxos = 0/0
debug_tp = 0/0
debug_auth = 0/0
debug_finisher = 0/0
debug_heartbeatmap = 0/0
debug_perfcounter = 0/0
debug_rgw = 0/0
debug_hadoop = 0/0
debug_asok = 0/0
debug_throttle = 0/0
rbd_default_format = 2

# Choose a reasonable crush leaf type
# 0 for a 1-node cluster.
# 1 for a multi node cluster in a single rack
# 2 for a multi node, multi chassis cluster with multiple hosts in a chassis
# 3 for a multi node cluster with hosts across racks, etc.
osd_crush_chooseleaf_type = 1

# Choose reasonable numbers for number of replicas and placement groups.
# Write an object 2 times
osd_pool_default_size = 2
# Allow writing 1 copy in a degraded state
osd_pool_default_min_size = 1 
osd_pool_default_pg_num = 256
osd_pool_default_pgp_num = 256

# --> Allow delete pool -- NOT RECOMMEND
mon_allow_pool_delete = false

# Journal size Jewel-release 
# osd journal size             = 20480    ; journal size, in megabytes

rbd_cache = true
bluestore_block_db_size = 5737418240
bluestore_block_wal_size = 2737418240

# Disable auto update crush => Modify Crushmap OSD tree  
osd_crush_update_on_start = false

# Backfilling and recovery
osd_max_backfills = 1
osd_recovery_max_active = 1
osd_recovery_max_single_start = 1
osd_recovery_op_priority = 1

# Osd recovery threads = 1
osd_backfill_scan_max = 16
osd_backfill_scan_min = 4
mon_osd_backfillfull_ratio = 0.95

# Scrubbing
osd_max_scrubs = 1
osd_scrub_during_recovery = false
# osd scrub begin hour = 22 
# osd scrub end hour = 4

# Max PG / OSD
mon_max_pg_per_osd = 500
```
