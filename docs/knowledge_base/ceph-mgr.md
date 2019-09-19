# Ceph MGR (Ceph Manager Daemon)

Ceph Manager (ceph-mgr) chạy song song cùng với hệ thống và làm nền giám sát chính, nhằm cung cấp hệ thống giám sát và giao diện bổ sung cho các hệ thống quản lý và giám sát bên ngoài.

Kể từ khi phát hành Ceph 12.x (Luminous), dịch vụ `ceph-mgr` được yêu cầu cài đặt luôn sau khi cài hệ thống, là một service cần thiết cho các hoạt động của cụm Ceph. 

Theo mặc định, ceph-mgr không yêu cầu cấu hình bổ sung, ngoài việc đảm bảo service mgr hoạt động. Nếu không có service mgr nào đang chạy,chúng ta sẽ thấy cảnh báo về HEALTH của cụm Ceph.

Ceph-mgr cung cấp các module bổ sung như `Dashboard`, `ResfullAPI`, `Zabbix`, `Prometheus` chủ yếu tập trung vào collect metrics toàn bộ cụm và có thể thao tác với cụm Ceph trên Dashboard(Từ bản Nautilous 14.2.x)

Ceph-mgr có thể triển khai dựa vào ceph-ansible, ceph-deploy...


# Tài liệu tham khảo 
- http://docs.ceph.com/docs/master/mgr/
