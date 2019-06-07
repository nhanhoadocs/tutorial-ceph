# Cài đặt check_mk



# Bổ sung agent check_mk check disk

```sh 
yum install smartmontools -y && sudo apt-get install smartmontools 
cd /usr/lib/check_mk_agent/plugins
wget https://raw.githubusercontent.com/nhanhoadocs/scripts/master/Utilities/smart
chmod +x smart
./smart
```

Kiểm tra bằng tay 
```sh 
sudo smartctl -i /dev/sda 
```

Discovery trên `check_mk`
![](../../../images/monitor/check_mk_disk.png)