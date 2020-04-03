## Thêm mới osd cho cluster ceph đã cài đặt trên ubuntu 18.04

## Tạo user "cephuser" trên ceph4
```sh
sudo useradd -m -s /bin/bash cephuser
passwd cephuser
echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser
sudo chmod 0440 /etc/sudoers.d/cephuser
sudo sed -i s'/Defaults requiretty/#Defaults requiretty'/g /etc/sudoers
```
## Lệnh bên dưới đều thao tác bằng user "cephuser", trên node ceph1
- Thêm node mới (ceph4) vào file hosts:
```
/etc/hots
```
- File quản lý clush:
```
/etc/clustershell/groups
```
## Copy id từ ceph1 lên ceph4 
```sh
ssh-copy-id ceph4
```
## Cài đặt python, ntp
```sh
ssh ceph4 "sudo apt install python ntp -y; timedatectl"
```
## Thay đổi hostname (nếu cần)
```sh
ssh ceph4 "hostnamectl set-hostname ceph4"
```
## Cài cmdlog
ssh ceph4 "curl -Lso- https://raw.githubusercontent.com/nhanhoadocs/scripts/master/Utilities/cmdlog.sh | sudo bash"

## Add key, resource
```sh
ssh ceph4 "wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add - ; sudo apt update -y"
ssh ceph4 "echo deb https://download.ceph.com/debian-nautilus/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list"
```

## Cài ceph lên node ceph4
```sh
ceph-deploy install --release nautilus ceph4

```
## Kiểm tra, so sánh, phiên bản
```sh
clush -a "ceph --version"
```

## Chỉ định, tạo osd trên ceph4
```sh
ceph-deploy disk zap ceph4 /dev/vdb
ceph-deploy disk zap ceph4 /dev/vdc
ceph-deploy osd create --data /dev/vdb ceph4
ceph-deploy osd create --data /dev/vdc ceph4
```

## Kiểm tra lại 
```sh
sudo ceph --status
sudo ceph osd tree
```
