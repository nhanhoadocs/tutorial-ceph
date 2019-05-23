# Ceph Object Storage - Ceph RadosGW 

## Chuẩn bị và môi trường LAB (2 Node)

- OS
- CentOS7 - 64 bit
- 03: HDD, trong đó:
- `sda`: sử dụng để cài OS
- `sdb`,`sdc`,: sử dụng làm OSD (nơi chứa dữ liệu)
- 03 NICs: 
- `eth0`: dùng để ssh và tải gói cài đặt
- `eth1`: dùng để các trao đổi thông tin giữa các node Ceph, cũng là đường Client kết nối vào
- `eth2`: dùng để đồng bộ dữ liệu giữa các OSD

- Host `cephaio` cài đặt `ceph-deploy`, `ceph-mon`,` ceph-osd`, `ceph-mgr`, `ceph-osd`

- Phiên bản cài đặt : Ceph luminous

- Mục tiêu: Tạo 1 node Ceph cung cấp dịch vụ Object và sử dụng Client kết nối vào

## Mô hình 
- Sử dụng mô hình

![](../../images/topo-aio.png)


## IP Planning
- Phân hoạch IP cho các máy chủ trong mô hình trên

![](../../images/ip-planning1.png)


## Các bước chuẩn bị trên từng Server

- Cài đặt NTPD 
```sh 
yum install chrony -y 
```

- Enable NTPD 
```sh 
systemctl start chronyd 
systemctl enable chronyd 
```

- Kiểm tra chronyd hoạt động 
```sh 
chronyc sources -v 
```

- Đặt hostname
```sh
hostnamectl set-hostname cephaio
```

- Đặt IP cho các node
```sh 
systemctl disable NetworkManager
systemctl stop NetworkManager
systemctl enable network
systemctl start network

echo "Setup IP eth0"
nmcli c modify eth0 ipv4.addresses 10.10.10.69/24
nmcli c modify eth0 ipv4.gateway 10.10.10.1
nmcli c modify eth0 ipv4.dns 8.8.8.8
nmcli c modify eth0 ipv4.method manual
nmcli con mod eth0 connection.autoconnect yes

echo "Setup IP eth1"
nmcli c modify eth1 ipv4.addresses 10.10.13.69/24
nmcli c modify eth1 ipv4.method manual
nmcli con mod eth1 connection.autoconnect yes

echo "Setup IP eth2"
nmcli c modify eth2 ipv4.addresses 10.10.14.69/24
nmcli c modify eth2 ipv4.method manual
nmcli con mod eth2 connection.autoconnect yes
```

- Cài đặt epel-relese và update OS 
```sh
yum install epel-release -y
yum update -y
```

- Cài đặt CMD_log 
```sh 
curl -Lso- https://raw.githubusercontent.com/nhanhoadocs/scripts/master/Utilities/cmdlog.sh | bash
```

- Vô hiệu hóa Selinux
```sh
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

- Mở port cho Ceph trên Firewalld  
```sh 
#ceph-admin
systemctl start firewalld
systemctl enable firewalld
sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
sudo firewall-cmd --zone=public --add-port=2003/tcp --permanent
sudo firewall-cmd --zone=public --add-port=4505-4506/tcp --permanent
sudo firewall-cmd --reload

# mon
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --zone=public --add-port=6789/tcp --permanent
sudo firewall-cmd --reload

# osd
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --zone=public --add-port=6800-7300/tcp --permanent
sudo firewall-cmd --reload
```

- Hoặc có thể disable firewall 
```sh 
sudo systemctl disable firewalld
sudo systemctl stop firewalld
```

- Bổ sung file hosts
```sh
cat << EOF >> /etc/hosts
10.10.13.69 cephaio
EOF
```
> Lưu ý network setup trong /etc/hosts chính là đường `eth1` dùng để các trao đổi thông tin giữa các node Ceph, cũng là đường Client kết nối vào

- Kiểm tra kết nối
```sh 
ping -c 10 cephaio
```

- Khởi động lại máy
```sh
init 6
```

## Cài đặt Ceph 

Các bước ở dưới được thực hiện toàn toàn trên Node `cephaio`

- Cài đặt `ceph-deploy`
```sh 
yum install -y wget 
wget https://download.ceph.com/rpm-luminous/el7/noarch/ceph-deploy-2.0.1-0.noarch.rpm --no-check-certificate
rpm -ivh ceph-deploy-2.0.1-0.noarch.rpm
```

- Cài đặt `python-setuptools` để `ceph-deploy` có thể hoạt động ổn định
```sh 
curl https://bootstrap.pypa.io/ez_setup.py | python
```

- Kiểm tra cài đặt 
```sh 
ceph-deploy --version
```
>Kết quả như sau là đã cài đặt thành công ceph-deploy
```sh 
2.0.1
```

- Tạo ssh key 
```sh
ssh-keygen
```
> Bấm ENTER khi có requirement 

- Copy ssh key sang các node khác
```sh
ssh-copy-id root@cephaio
```

- Tạo các thư mục `ceph-deploy` để thao tác cài đặt vận hành Cluster
```sh
mkdir /ceph-deploy && cd /ceph-deploy
```

- Khởi tại file cấu hình cho cụm với node quản lý là `cephaio`
```sh
ceph-deploy new cephaio
```

- Kiểm tra lại thông tin folder `ceph-deploy`
```sh 
[root@cephaio ceph-deploy]# ls -lah
total 12K
drwxr-xr-x   2 root root   75 Jan 31 16:31 .
dr-xr-xr-x. 18 root root  243 Jan 31 16:29 ..
-rw-r--r--   1 root root 2.9K Jan 31 16:31 ceph-deploy-ceph.log
-rw-r--r--   1 root root  195 Jan 31 16:31 ceph.conf
-rw-------   1 root root   73 Jan 31 16:31 ceph.mon.keyring
[root@cephaio ceph-deploy]#
```
- `ceph.conf` : file config được tự động khởi tạo
- `ceph-deploy-ceph.log` : file log của toàn bộ thao tác đối với việc sử dụng lệnh `ceph-deploy`
- `ceph.mon.keyring` : Key monitoring được ceph sinh ra tự động để khởi tạo Cluster

- Chúng ta sẽ bổ sung thêm vào file `ceph.conf` một vài thông tin cơ bản như sau:
```sh
cat << EOF >> /ceph-deploy/ceph.conf
osd pool default size = 2
osd pool default min size = 1
osd crush chooseleaf type = 0
osd pool default pg num = 128
osd pool default pgp num = 128

public network = 10.10.13.0/24
cluster network = 10.10.14.0/24

mon_max_pg_per_osd = 500
EOF
```
- Bổ sung thêm định nghĩa 
    + `public network` : Đường trao đổi thông tin giữa các node Ceph và cũng là đường client kết nối vào 
    + `cluster network` : Đường đồng bộ dữ liệu
- Bổ sung thêm `default size replicate`
- Bổ sung thêm `default pg num`


- Cài đặt ceph trên toàn bộ các node ceph
```sh
ceph-deploy install --release luminous cephaio
```

- Kiểm tra sau khi cài đặt 
```sh 
ceph -v 
```
> Kết quả như sau là đã cài đặt thành công ceph trên node 
```sh 
ceph version 12.2.9 (9e300932ef8a8916fb3fda78c58691a6ab0f4217) luminous (stable)
```

- Khởi tạo cluster với các node `mon` (Monitor-quản lý) dựa trên file `ceph.conf`
```sh
ceph-deploy mon create-initial
```

- Sau khi thực hiện lệnh phía trên sẽ sinh thêm ra 05 file : `ceph.bootstrap-mds.keyring`, `ceph.bootstrap-mgr.keyring`, `ceph.bootstrap-osd.keyring`, `ceph.client.admin.keyring` và `ceph.bootstrap-rgw.keyring`. Quan sát bằng lệnh `ll -alh`

```sh
[root@cephaio ceph-deploy]# ls -lah
total 348K
drwxr-xr-x   2 root root  244 Feb  1 11:40 .
dr-xr-xr-x. 18 root root  243 Feb  1 11:29 ..
-rw-r--r--   1 root root 258K Feb  1 11:40 ceph-deploy-ceph.log
-rw-------   1 root root  113 Feb  1 11:40 ceph.bootstrap-mds.keyring
-rw-------   1 root root  113 Feb  1 11:40 ceph.bootstrap-mgr.keyring
-rw-------   1 root root  113 Feb  1 11:40 ceph.bootstrap-osd.keyring
-rw-------   1 root root  113 Feb  1 11:40 ceph.bootstrap-rgw.keyring
-rw-------   1 root root  151 Feb  1 11:40 ceph.client.admin.keyring
-rw-r--r--   1 root root  195 Feb  1 11:29 ceph.conf
-rw-------   1 root root   73 Feb  1 11:29 ceph.mon.keyring
```

- Để node `cephaio` có thể thao tác với cluster chúng ta cần gán cho node `cephaio` với quyền admin bằng cách bổ sung cho node này `admin.keying`
```sh  
ceph-deploy admin cephaio
```
> Kiểm tra bằng lệnh 
```sh
[root@cephaio ceph-deploy]# ceph -s
cluster:
    id:     39d1a369-bf54-8907-d49b-490a771ac0e2
    health: HEALTH_OK

services:
    mon: 1 daemons, quorum cephaio
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in

data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

## Khởi tạo MGR

Ceph-mgr là thành phần cài đặt yêu cầu cần khởi tạo từ bản Luminous, có thể cài đặt trên nhiều node hoạt động theo cơ chế `Active-Passive`

- Cài đặt ceph-mgr trên cephaio
```sh
ceph-deploy mgr create cephaio
```

- Kiểm tra cài đặt 
```sh
[root@cephaio ceph-deploy]# ceph -s
cluster:
    id:     39d1a369-bf54-8907-d49b-490a771ac0e2
    health: HEALTH_OK

services:
    mon: 1 daemons, quorum cephaio
    mgr: cephaio(active)
    osd: 0 osds: 0 up, 0 in

    data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

- Ceph-mgr hỗ trợ dashboard để quan sát trạng thái của cluster, Enable mgr dashboard trên host cephaio

```sh
ceph mgr module enable dashboard
ceph mgr services
```

- Truy cập vào mgr dashboard với username và password vừa đặt ở phía trên để kiểm tra
```sh 
https://<ip-cephaio>:7000
```
![](../../images/dashboard-l.png)


## Khởi tạo OSD

Tạo OSD thông qua ceph-deploy tại host cephaio

- Trên cephaio, dùng ceph-deploy để partition ổ cứng OSD, thay `cephaio` bằng hostname của host chứa OSD
```sh
ceph-deploy disk zap cephaio /dev/sdb
```

- Tạo OSD với ceph-deploy
```sh
ceph-deploy osd create --data /dev/sdb cephaio
``` 

- Kiểm tra osd vừa tạo bằng lệnh
```sh
ceph osd tree
``` 

- Kiểm tra ID của OSD bằng lệnh
```sh
lsblk
```

Kết quả:
```sh
sdb                                                                                                     8:112  0   39G  0 disk  
└─ceph--42804049--4734--4a87--b776--bfad5d382114-osd--data--e6346e12--c312--4ccf--9b5f--0efeb61d0144  253:5    0   39G  0 lvm   /var/lib/ceph/osd/ceph-0
```

## Kiểm tra
Thực hiện trên cephaio
- Kiểm tra trạng thái của CEPH sau khi cài
```sh
ceph -s
```

- Kết quả của lệnh trên như sau: 
```sh
ceph-deploy@cephaio:~/my-cluster$ ceph -s
cluster:
    id:     39d1a369-bf54-8907-d49b-490a771ac0e2
    health: HEALTH_OK

services:
    mon: 1 daemons, quorum cephaio
    mgr: cephaio(active)
    osd: 2 osds: 2 up, 2 in

data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 bytes
    usage:   3180 MB used, 116 GB / 119 GB avail
    pgs:     
```

- Nếu có dòng `health HEALTH_OK` thì việc cài đặt đã ok.

- Kiểm tra rule và OSD đảm bảo việc replicate trên HOST
```sh
[root@cephaio ~]# ceph osd crush rule dump
[
    {
        "rule_id": 0,
        "rule_name": "replicated_rule",
        "ruleset": 0,
        "type": 1,
        "min_size": 1,
        "max_size": 10,
        "steps": [
            {
                "op": "take",
                "item": -1,
                "item_name": "default"
            },
            {
                "op": "choose_firstn",
                "num": 0,
                "type": "osd"
            },
            {
                "op": "emit"
            }
        ]
    }
]

[root@cephaio ~]# 
```


Dump và chỉnh sửa crushmap
```sh
ceph osd getcrushmap -o crushmap
crushtool -d crushmap -o crushmap.decom
sed -i 's|step choose firstn 0 type osd|step chooseleaf firstn 0 type osd|g' crushmap.decom
crushtool -c crushmap.decom -o crushmap.new
ceph osd setcrushmap -i crushmap.new
```

Kiểm tra lại rules 
```sh
[root@cephaio ceph-deploy]# ceph osd crush rule dump
[
    {
        "rule_id": 0,
        "rule_name": "replicated_rule",
        "ruleset": 0,
        "type": 1,
        "min_size": 1,
        "max_size": 10,
        "steps": [
            {
                "op": "take",
                "item": -1,
                "item_name": "default"
            },
            {
                "op": "chooseleaf_firstn",
                "num": 0,
                "type": "osd"
            },
            {
                "op": "emit"
            }
        ]
    }
]

[root@cephaio ceph-deploy]# 
```

# Create pool 
Truy cập trang ceph.com/pgcalc để tính toán create các pool 
![](http://i.imgur.com/suH6zuL.png)

```
## Note: The 'while' loops below pause between pools to allow all
##       PGs to be created.  This is a safety mechanism to prevent
##       saturating the Monitor nodes.
## -------------------------------------------------------------------

ceph osd pool create .rgw.root 2
ceph osd pool set .rgw.root size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done

ceph osd pool create default.rgw.control 2
ceph osd pool set default.rgw.control size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done

ceph osd pool create default.rgw.data.root 2
ceph osd pool set default.rgw.data.root size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done

ceph osd pool create default.rgw.gc 2
ceph osd pool set default.rgw.gc size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done

ceph osd pool create default.rgw.log 2
ceph osd pool set default.rgw.log size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done

ceph osd pool create default.rgw.intent-log 2
ceph osd pool set default.rgw.intent-log size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done

ceph osd pool create default.rgw.meta 2
ceph osd pool set default.rgw.meta size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done

ceph osd pool create default.rgw.usage 2
ceph osd pool set default.rgw.usage size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done

ceph osd pool create default.rgw.users.keys 2
ceph osd pool set default.rgw.users.keys size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done

ceph osd pool create default.rgw.users.email 2
ceph osd pool set default.rgw.users.email size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done

ceph osd pool create default.rgw.users.swift 2
ceph osd pool set default.rgw.users.swift size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done

ceph osd pool create default.rgw.users.uid 2
ceph osd pool set default.rgw.users.uid size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done

ceph osd pool create default.rgw.buckets.extra 2
ceph osd pool set default.rgw.buckets.extra size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done

ceph osd pool create default.rgw.buckets.index 8
ceph osd pool set default.rgw.buckets.index size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done

ceph osd pool create default.rgw.buckets.data 256
ceph osd pool set default.rgw.buckets.data size 2
while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done
```

Kiểm tra các pool được tạo
```sh 
[root@cephaio ~]# ceph df 
GLOBAL:
    SIZE        AVAIL       RAW USED     %RAW USED 
    40.0GiB     38.0GiB      2.01GiB          5.03 
POOLS:
    NAME                          ID     USED     %USED     MAX AVAIL     OBJECTS 
    .rgw.root                     1        0B         0       18.0GiB           0 
    default.rgw.control           2        0B         0       18.0GiB           0 
    default.rgw.data.root         3        0B         0       18.0GiB           0 
    default.rgw.gc                4        0B         0       18.0GiB           0 
    default.rgw.log               5        0B         0       18.0GiB           0 
    default.rgw.intent-log        6        0B         0       18.0GiB           0 
    default.rgw.meta              7        0B         0       18.0GiB           0 
    default.rgw.usage             8        0B         0       18.0GiB           0 
    default.rgw.users.keys        9        0B         0       18.0GiB           0 
    default.rgw.users.email       10       0B         0       18.0GiB           0 
    default.rgw.users.swift       11       0B         0       18.0GiB           0 
    default.rgw.users.uid         12       0B         0       18.0GiB           0 
    default.rgw.buckets.extra     13       0B         0       18.0GiB           0 
    default.rgw.buckets.index     14       0B         0       18.0GiB           0 
    default.rgw.buckets.data      15       0B         0       18.0GiB           0 
[root@cephaio ~]# 
```
Kiểm tra 
![](http://i.imgur.com/GHoIufT.png)

Hoặc kiểm tra trên Terminal 

Chưa OK 
![](http://i.imgur.com/xd2pbK0.png)

Khởi động lại 

OK 
![](http://i.imgur.com/7hEdD2r.png)


> Yêu cầu phải nếu trên cùng 1 host
```sh 
                "op": "chooseleaf_firstn",
                "num": 0,
                "type": "osd"
```

Cài đặt Ceph RadosGW 
http://docs.ceph.com/docs/mimic/install/install-ceph-gateway/


Khởi tạo node `rgw`
```sh 
cd /ceph-deploy
ceph-deploy install --rgw cephaio 
```

> 2019-05-22 Stable RadosGW 
```sh 
[ceph_deploy.install][DEBUG ] Installing stable version mimic on cluster ceph hosts cephaio
```

```
[cephaio][INFO  ] Running command: ceph --version
[cephaio][DEBUG ] ceph version 13.2.5 (cbff874f9007f1869bfd3821b7e33b2a6ffd4988) mimic (stable)
```

```sh
[root@cephaio ceph-deploy]# ceph -s 
  cluster:
    id:     7341daaf-82e4-4de0-bf90-3f6616cb415d
    health: HEALTH_OK
 
  services:
    mon: 1 daemons, quorum cephaio
    mgr: cephaio(active)
    osd: 2 osds: 2 up, 2 in
 
  data:
    pools:   15 pools, 290 pgs
    objects: 0 objects, 0B
    usage:   2.01GiB used, 38.0GiB / 40.0GiB avail
    pgs:     290 active+clean
 
[root@cephaio ceph-deploy]#
```

```
[root@cephaio ceph-deploy]# ceph -s 
  cluster:
    id:     7341daaf-82e4-4de0-bf90-3f6616cb415d
    health: HEALTH_OK
 
  services:
    mon: 1 daemons, quorum cephaio
    mgr: cephaio(active)
    osd: 2 osds: 2 up, 2 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0B
    usage:   2.00GiB used, 38.0GiB / 40.0GiB avail
    pgs:     
 
[root@cephaio ceph-deploy]# ceph mgr services 
{
    "dashboard": "http://cephaio:7000/"
}
[root@cephaio ceph-deploy]# init 6 
```

Khởi động lại node 


Enable lại dashboard
```sh 
ceph mgr module enable dashboard
ceph dashboard create-self-signed-cert
ceph dashboard set-login-credentials <username> <password>
ceph mgr services
```

Kiểm tra dashboard
```sh 
[root@cephaio ~]# systemctl restart ceph-mgr@cephaio 
[root@cephaio ~]# ceph mgr services
{}
[root@cephaio ~]# ceph mgr services
{
    "dashboard": "https://cephaio:8443/"
}
[root@cephaio ~]# 
```

Kết nối Client Ceph 
http://lists.ceph.com/pipermail/ceph-users-ceph.com/2013-May/020776.html




