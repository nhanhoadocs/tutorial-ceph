## Hướng dẫn cài đặt CEPH sử dụng `ceph-deploy`

### Mục tiêu LAB
- Mô hình này sử dụng 3 server, trong đó:
- Host `ceph01` cài đặt `ceph-deploy`, `ceph-mon`,` ceph-osd`, `ceph-mgr`
- Host `ceph02` cài đặt `ceph-osd`
- Host `ceph03` cài đặt `ceph-osd`
- Mô hình khá cơ bản cho việc áp dụng vào môi trường Product

## Chuẩn bị và môi trường LAB (3 Node)

- Ubuntu 18.04.4 - 64 bit
- 03: HDD, trong đó:
    - `sda`: sử dụng để cài OS
    - `vdb`,`vdc`: sử dụng làm OSD (nơi chứa dữ liệu)
- 03 NICs: 
    - `eth0`: dùng để ssh và tải gói cài đặt
    - `eth1`: dùng để các trao đổi thông tin giữa các node Ceph, cũng là đường Client kết nối vào
    - `eth2`: dùng để đồng bộ dữ liệu giữa các OSD
- Phiên bản cài đặt : Ceph Nautilus


## Mô hình 
- Sử dụng mô hình

![](../../images/topo.png)


## IP Planning
- Phân hoạch IP cho các máy chủ trong mô hình trên

![](../../images/ip-planning1.png)

## Bổ sung file hosts
```sh
cat << EOF >> /etc/hosts
10.10.13.61 ceph01
10.10.13.62 ceph02
10.10.13.63 ceph03
EOF
```
> Lưu ý network setup trong /etc/hosts chính là đường `eth1` dùng để các trao đổi thông tin giữa các node Ceph, cũng là đường Client kết nối vào

## Cài đặt clushter shell
```sh 
sudo apt install clustershell -y
echo 'all: ceph[01-03]' > /etc/clustershell/groups
```

## Tạo user "cephuser" trên tất cả các node
```sh
sudo useradd -m -s /bin/bash cephuser
sudo passwd cephuser
echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser
sudo chmod 0440 /etc/sudoers.d/cephuser
sudo sed -i s'/Defaults requiretty/#Defaults requiretty'/g /etc/sudoers

su - cephuser
ssh-keygen

for i in {1..3}; do ssh-copy-id ceph$i ; done
```
## Các bước chuẩn bị trên từng Server

- Cài đặt Python, ntpdate
```sh 
clush -a "sudo apt update && sudo apt install ntpdate python -y"
```

- Cập nhật thời gian
```sh 
clush -a "sudo ntpdate 1.ro.pool.ntp.org"
clush -a "timedatectl"
```

- Set hwclock 
```sh 
clush -a "sudo hwclock --systohc"
```

- Đặt hostname (chạy lệnh trên từng node)
```sh
hostnamectl set-hostname ceph01
```

- Cài đặt CMD_log 
```sh 
clush -a "curl -Lso- https://raw.githubusercontent.com/nhanhoadocs/scripts/master/Utilities/cmdlog.sh | sudo bash"
```

Hoặc
```sh
#!/bin/bash
echo "Install rsyslog"
yum -y install rsyslog || apt-get -y install rsyslog 
systemctl enable rsyslog.service || chkconfig rsyslog on
systemctl start rsyslog.service || service rsyslog start

echo "Config log History"
touch ~/.bash_profile
cp ~/{.bash_profile,.bash_profile.bk}
echo "export PROMPT_COMMAND='RETRN_VAL=$?;logger -p local6.debug \"[\$(echo \$SSH_CLIENT | cut -d\" \" -f1)] # \$(history 1 | sed \"s/^[ ]*[0-9]\+[ ]*//\" )\"'" >> ~/.bash_profile
echo 'export HISTTIMEFORMAT="%d/%m/%y %T "' >> ~/.bash_profile
touch /var/log/cmdlog.log

echo "Config rsyslog"
mv /etc/rsyslog.{conf,conf.bk}
cat >> /etc/rsyslog.conf << EOF
\$ModLoad imuxsock
\$ModLoad imklog
\$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
\$FileOwner root
\$FileGroup adm
\$FileCreateMode 0640
\$DirCreateMode 0755
\$Umask 0022
auth,authpriv.*-/var/log/auth.log
daemon.*-/var/log/daemon.log
kern.*-/var/log/kern.log
cron.*-/var/log/cron.log
user.*-/var/log/user.log
mail.*-/var/log/mail.log
local7.*-/var/log/boot.log
local6.*-/var/log/cmdlog.log
EOF

echo "Restart rsyslog"
systemctl restart  rsyslog.service || service rsyslog restart
source ~/.bash_profile
```

- Khởi động lại máy
```sh
init 6
```

## Cài đặt Ceph 

Bổ sung repo cho ceph trên tất cả các node
```sh 
clush -a "curl -Lso- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -"
clush -a "echo deb https://download.ceph.com/debian-octopus/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list"
clush -a "sudo apt update -y"
```

Các bước ở dưới được thực hiện toàn toàn trên Node `ceph01`

- Cài đặt `ceph-deploy`
```sh 
sudo apt -y install ceph-deploy 
```

- Kiểm tra cài đặt 
```sh 
ceph-deploy --version
```

- Cấu hình user ssh cho ceph-deploy
```sh 
cat <<EOF> ~/.ssh/config
Host ceph01
    Hostname ceph01
    User cephuser
Host ceph02
    Hostname ceph02
    User cephuser
Host ceph03
    Hostname ceph03
    User cephuser
EOF
```

- Tạo các thư mục `ceph-deploy` để thao tác cài đặt vận hành Cluster
```sh
mkdir ~/ceph-deploy ; cd ~/ceph-deploy
```

- Khởi tại file cấu hình cho cụm với node quản lý là `ceph01`
```sh
ceph-deploy new ceph01 ceph02 ceph03
```

- Kiểm tra lại thông tin folder `ceph-deploy`
```sh 
[cephuser@ceph01 ceph-deploy]$ ls -lah
total 12K
drwxr-xr-x   2 cephuser cephuser  75 Jan 31 16:31 .
dr-xr-xr-x. 18 cephuser cephuser 243 Jan 31 16:29 ..
-rw-r--r--   1 cephuser cephuser2.9K Jan 31 16:31 ceph-deploy-ceph.log
-rw-r--r--   1 cephuser cephuser 195 Jan 31 16:31 ceph.conf
-rw-------   1 cephuser cephuser  73 Jan 31 16:31 ceph.mon.keyring
[cephuser@ceph01 ceph-deploy]$
```
- `ceph.conf` : file config được tự động khởi tạo
- `ceph-deploy-ceph.log` : file log của toàn bộ thao tác đối với việc sử dụng lệnh `ceph-deploy`
- `ceph.mon.keyring` : Key monitoring được ceph sinh ra tự động để khởi tạo Cluster

- Chúng ta sẽ bổ sung thêm vào file `ceph.conf` một vài thông tin cơ bản như sau:
```sh
cat << EOF >> ceph.conf
osd pool default size = 2
osd pool default min size = 1
osd pool default pg num = 128
osd pool default pgp num = 128

osd crush chooseleaf type = 1

public network = 10.10.13.0/24
cluster network = 10.10.14.0/24
EOF
```
- Bổ sung thêm định nghĩa 
    + `public network` : Đường trao đổi thông tin giữa các node Ceph và cũng là đường client kết nối vào 
    + `cluster network` : Đường đồng bộ dữ liệu
- Bổ sung thêm `default size replicate`
- Bổ sung thêm `default pg num`


- Cài đặt ceph trên toàn bộ các node ceph
```sh
ceph-deploy install --release octopus ceph01 ceph02 ceph03 
```

- Kiểm tra sau khi cài đặt trên cả 3 node
```sh 
clush -a "ceph -v"
```

- Khởi tạo cluster với các node `mon` (Monitor-quản lý) dựa trên file `ceph.conf`
```sh
ceph-deploy mon create-initial
```

- Sau khi thực hiện lệnh phía trên sẽ sinh thêm ra 05 file : 
`ceph.bootstrap-mds.keyring`, `ceph.bootstrap-mgr.keyring`, `ceph.bootstrap-osd.keyring`, `ceph.client.admin.keyring` và `ceph.bootstrap-rgw.keyring`.

```sh
[cephuser@ceph01 ceph-deploy]# ls -lah
total 348K
drwxr-xr-x   2 cephuser cephuser 244 Feb  1 11:40 .
dr-xr-xr-x. 18 cephuser cephuser 243 Feb  1 11:29 ..
-rw-r--r--   1 cephuser cephuser258K Feb  1 11:40 ceph-deploy-ceph.log
-rw-------   1 cephuser cephuser 113 Feb  1 11:40 ceph.bootstrap-mds.keyring
-rw-------   1 cephuser cephuser 113 Feb  1 11:40 ceph.bootstrap-mgr.keyring
-rw-------   1 cephuser cephuser 113 Feb  1 11:40 ceph.bootstrap-osd.keyring
-rw-------   1 cephuser cephuser 113 Feb  1 11:40 ceph.bootstrap-rgw.keyring
-rw-------   1 cephuser cephuser 151 Feb  1 11:40 ceph.client.admin.keyring
-rw-r--r--   1 cephuser cephuser 195 Feb  1 11:29 ceph.conf
-rw-------   1 cephuser cephuser  73 Feb  1 11:29 ceph.mon.keyring
```

- Để node `ceph01`, `ceph02`, `ceph03` có thể thao tác với cluster chúng ta cần gán cho node quyền admin bằng cách bổ sung key `admin.keying` cho node
```sh  
ceph-deploy admin ceph01 ceph02 ceph03
```
> Kiểm tra bằng lệnh 
```sh
[cephuser@ceph01 ceph-deploy]# sudo ceph -s 
  cluster:
    id:     ba7c7fa1-4e55-450b-bc40-4cf122b28c27
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph01,ceph02,ceph03 (age 1.09337s)
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:     
```

## Khởi tạo MGR

Ceph-mgr là thành phần cài đặt yêu cầu cần khởi tạo từ bản luminous, có thể cài đặt trên nhiều node hoạt động theo cơ chế `Active-Passive`

- Cài đặt ceph-mgr trên ceph01
```sh
ceph-deploy mgr create ceph01 ceph02
```

- Kiểm tra cài đặt 
```sh
[cephuser@ceph01 ceph-deploy]# sudo ceph -s 
  cluster:
    id:     ba7c7fa1-4e55-450b-bc40-4cf122b28c27
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph01,ceph02,ceph03 (age 53s)
    mgr: ceph01(active, since 11s), standbys: ceph02
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:     
```

- Ceph-mgr hỗ trợ dashboard để quan sát trạng thái của cluster, Enable mgr dashboard trên host ceph01
> Sẽ cài đặt `ceph-mgr-dashboard` trên cả `ceph01` và `ceph02` 

```sh
clush -w ceph[01,02] "sudo apt install ceph-mgr-dashboard -y"
sudo ceph mgr module enable dashboard
# Nếu có lỗi "...No module named 'distutils.util' (pass --force to force enablement)", hãy cài "sudo apt-get install python-distutils-extra"
sudo ceph dashboard create-self-signed-cert
sudo ceph dashboard ac-user-create <username> <password> administrator
sudo ceph mgr services
```


- Truy cập vào mgr dashboard với username và password vừa đặt ở phía trên để kiểm tra
```sh 
https://<ip-ceph01>:8443
```
![](../../images/dashboard-o.jpg)


## Khởi tạo OSD

Tạo OSD thông qua ceph-deploy tại host `ceph01`

- Thực hiện zapdisk 
```sh
ceph-deploy disk zap ceph01 /dev/vdb
ceph-deploy disk zap ceph01 /dev/vdc
```

- Tạo OSD với ceph-deploy
```sh
ceph-deploy osd create --data /dev/vdb ceph01
ceph-deploy osd create --data /dev/vdc ceph01
``` 

- Thao tác với các ổ trên `ceph02` và `ceph03` tương tự. Vẫn thực hiện trên thư mục `ceph-deploy` trên `ceph01`
```sh 
# ceph02 
ceph-deploy disk zap ceph02 /dev/vdb
ceph-deploy disk zap ceph02 /dev/vdc
ceph-deploy osd create --data /dev/vdb ceph02
ceph-deploy osd create --data /dev/vdc ceph02

# ceph03
ceph-deploy disk zap ceph03 /dev/vdb
ceph-deploy disk zap ceph03 /dev/vdc
ceph-deploy osd create --data /dev/vdb ceph03
ceph-deploy osd create --data /dev/vdc ceph03
```

- Kiểm tra osd vừa tạo bằng lệnh
```sh
ceph osd tree
``` 
- Kết quả 
```sh 
[cephuser@ceph01 ceph-deploy]$ ceph osd tree 
ID CLASS WEIGHT  TYPE NAME       STATUS REWEIGHT PRI-AFF 
-1       0.16974 root default                            
-3       0.05658     host ceph01                         
 0   hdd 0.02829         osd.0       up  1.00000 1.00000 
 1   hdd 0.02829         osd.1       up  1.00000 1.00000 
-5       0.05658     host ceph02                         
 2   hdd 0.02829         osd.2       up  1.00000 1.00000 
 3   hdd 0.02829         osd.3       up  1.00000 1.00000 
-7       0.05658     host ceph03                         
 4   hdd 0.02829         osd.4       up  1.00000 1.00000 
 5   hdd 0.02829         osd.5       up  1.00000 1.00000 
```

## Kiểm tra
Thực hiện trên ceph01
- Kiểm tra trạng thái của CEPH sau khi cài
```sh
ceph -s
```

- Kết quả của lệnh trên như sau: 
```sh
[cephuser@ceph01 ceph-deploy]$ ceph -s
  cluster:
    id:     ba7c7fa1-4e55-450b-bc40-4cf122b28c27
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph01,ceph02,ceph03 (age 10m)
    mgr: ceph01(active, since 7m), standbys: ceph02
    osd: 6 osds: 6 up (since 51s), 6 in (since 51s)
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   6.0 GiB used, 168 GiB / 174 GiB avail
    pgs:          
```

- Nếu có dòng `health HEALTH_OK` thì việc cài đặt đã thành công.
