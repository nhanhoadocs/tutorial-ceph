# Cài đặt 

```sh 
yum install wwget -y 
wget https://dl.influxdata.com/influxdb/releases/influxdb-1.7.8.x86_64.rpm
sudo yum localinstall influxdb-1.7.8.x86_64.rpm


systemctl enable --now influxdb


firewall-cmd --permanent --add-port=8086/tcp
firewall-cmd --permanent --add-port=8088/tcp
firewall-cmd --reload
```

# Tài liệu tham khảo 
[](https://computingforgeeks.com/how-to-install-influxdb-on-rhel-8-centos-8/)

