# Ceph Monitor (MON)

Ceph monitor chịu trách nhiệm giám sát tình trạng của toàn hệ thống. Nó hoạt động như các daemon duy trì sự kết nối trong cluster bằng cách chứa các thông tin cơ bản về cluster, tình trạng các node lưu trữ và thông tin cấu hình cluster. Ceph monitor thực hiện điều này bằng cách duy trì các cluster map. Các cluster map này bao gồm monitor, OSD, PG, CRUSH và MDS map.

## 1. Các map trong MON
- **Monitor map**: map này lưu giữ thông tin về các node monitor, gồm CEPH Cluster ID, monitor hostname, địa chỉ IP và số port. Nó cũng giữ epoch (phiên bản map tại một thời điểm) hiện tại để tạo map và thông tin về lần thay đổi map cuối cùng. Có thể kiểm tra bằng câu lệnh:
```
ceph mon dump
```

- **OSD map**: map này lưu giữ các trường như cluster ID, epoch cho việc tạo map OSD và lần thay đổi cuối., và thông tin liên quan đến pool như tên, ID, loại, mức nhân bản và PG. Nó cũng lưu các thông tin OSD như tình trạng, trọng số, thông tin host OSD. Có thể kiểm tra OSD map bằng câu lệnh:
```
ceph osd dump
```

- **PG map**: map này lưu giữ các phiên bản của PG (thành phần quản lý các object trong ceph), timestamp, bản OSD map cuối cùng, tỉ lệ đầy và gần đầy dung lượng. Nó cũng lưu các ID của PG, object count, tình trạng hoạt động và srub (hoạt động kiểm tra tính nhất quán của dữ liệu lưu trữ). Kiểm tra PG map bằng lệnh:
```
ceph pg dump
```

- **CRUSH map**: map này lưu các thông tin của các thiết bị lưu trữ trong Cluster, các rule cho tưng vùng lưu trữ. Kiểm tra CRUSH map bằng lệnh:
```
ceph osd crush dump
```

- **MDS map**: lưu thông tin về thời gian tạo và chỉnh sửa, dữ liệu và metadata pool ID, cluster MDS count, tình trạng hoạt động của MDS, epoch của MDS map hiện tại. Kiểm tra MDS map bằng lệnh:
```
ceph mds dump
```

## 2. Hướng triển khai MON
- Ceph monitor không lưu trữ dữ liệu, thay vào đó, nó gửi các bản update cluster map cho client và các node khác trong cluster. Client và các node khác định kỳ check các cluster map với monitor node.

- Monitor là lightweight daemon không yêu cầu nhiều tài nguyên tính toán. Một Server thuộc dòng entry-server, CPU, RAM vừa phải và card mạng 1 GbE là đủ. Monitor node cần có đủ không gian lưu trữ để chứa các cluster logs, gồm OSD, MDS và monitor log. Một cluster ổn định sinh ra lượng log từ vài MB đến vài GB. Tuy nhiên, dung lượng cho log có thể tăng lên khi mức độ debug bị thay đổi, có thể lên đến một vài GB.

- Cần đặt ra các policy cho log và các giải pháp monitor dung lượng lưu trữ, trên các node monitor tăng mức debug, lượng log có thể lên tới 1GB/1h.

- Mô hình Ceph nhiều node Mon đưa ra khái niệm quorum bằng cách sử dụng thuật toán [PAXOS](paxos.md). Số monitor trong hệ thống nên là số lẻ, ít nhất là 1 node, và khuyến nghị là 3 node. VÌ hoạt động trong quorum, hơn một nửa số lượng monitor node cần đảm bảo sẵn sàng để tránh rơi vào trạng thái "split-brain". Một monitor node sẽ là leader, và các node còn lại sẽ đưa đưa lên làm leader nếu node ban đầu bị lỗi.

- Nếu hạn chế về tài chính, monitor daemon có thể chạy cùng trên OSD node. Tuy nhiên, cần trang bị nhiều CPU, RAM và ổ cứng hơn để lưu monitor logs.

- Đối với các hệ thống lớn, nên sử dụng node monitor chuyên dụng. Đặt các node montior trên các rack riêng biệt với switch và nguồn điện riêng biệt. Nếu có nhiều datacenter với đường mạng tốc độ cao, có thể đặt monitor node trên nhiều DC.

- Các node monitor thường khuyến cáo sử dụng ổ SSD để lưu trữ phân vùng `/var/lib/ceph/ceph-mon` nhằm tăng tốc độ xử lý cho cụm

## 3. Monitor Commands

- Kiểm tra trạng thái service, dùng câu lệnh:
```
service ceph status mon
```

- Các câu lệnh đẻ kiểm tra trạng thái các node monitor:
```
ceph mon stat
ceph mon_status
ceph mon dump
```