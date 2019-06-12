# Ceph Overview 

Ceph là một phần mềm mã nguồn mở với cộng đồng sử dụng và phát triển rất đông. Nó được thiết kế để chạy trên các nền tảng phần cứng thông thường, với quan điểm thiết kế các hệ thống lưu trữ – mở rộng đến hàng petabyte nhưng với chi phí rẻ, hợp lý.

Hệ thống lưu trữ Ceph có thể được sử dụng với các mục đích khác nhau và cung cấp nhiều hình thức lưu trữ như: 
- Ceph Object Storage, 
- Ceph Block Storage 
- Ceph FileSystem, hoặc cho bất kỳ mục đích nào liên quan đến lưu trữ.

> Object và Block của Ceph thường thấy trong các nền tảng điện toán đám mây như OpenStack 

Việc Ceph phổ biến là do Ceph là một 
Một Cluster Ceph được tạo thành từ việc kết nối các node Ceph lại với nhau nó cần ít nhất 1 Ceph Monitor và 2 Ceph OSD Daemons. 

Ceph OSDs: Ceph OSD daemon (Ceph OSD) lưu trữ data, xử lý việc đồng bộ dữ liệu, recovery, rebalancing và cung cấp thông tin liên quan đến monitoring đến cho Ceph monitoring bằng cách kiểm tra các Ceph OSD daemons khác thông qua heartbeat. Một ceph storage cluster cần ít nhất 2 ceph OSD daemons để hướng tới trạng thái active + clean, lúc này hệ thống sẽ có 02 bản copy của data (default của Ceph là 3 bản).

Monitors: ceph monitor sẽ theo dõi trạng thái của cluster, bao gồm việc theo dõi các monitor map, OSD map, placement group (PG) map, và CRUSH map. Ceph lưu thông tin lịch sử (trong ceph gọi là “epoch” của mỗi trạng thái thay đổi của Ceph Monitors, Ceph OSD Daemons, và PGs.

MDSs: một Ceph Metadata Server (MDS) lưu trữ thông tin về metadata của hệ thống Ceph FileSystem (ceph block device và object storage không sử dụng MDS).

Ceph lưu trữ dữ liệu của client dưới dạng các objects trong các pool lưu trữ. Ceph sử dụng thuật toán CRUSH, trong đó Ceph sẽ tính toán placement group nào sẽ lưu trữ object, và tính toán Ceph OSD Daemon nào sẽ lưu trữ placement group. CRUSH algorithm cho phép Ceph Storage cluster khả năng mở rộng, tự cân bằng (rebalance), và recovery tự động.

CPU
Ceph MDS cần nhiều CPU hơn các thành phần khác trong hệ thống, khuyến cáo sử dụng quad core hoặc CPU tốt hơn. Ceph OSDs chạy RADOS service, tính toán data placement với thuật toán CRUSH, đồng bộ dữ liệu, và duy trì bản copy của cluster map. Vì vậy, OSDs cũng cần 1 lượng CPU nhất định, khuyến nghị sử dụng dual core processors. Monitor hoạt động không cần nhiều CPU. Cần lưu ý trong trường hợp máy chủ chạy dịch vụ tính toán trên các OSD, ví dụ mô hình hoạt động của OpenStack – kết hợp giữa compute node và storage node, cần thiết kế CPU để bảo đảm đủ năng lực dành riêng cho Ceph daemons. Khuyến cáo nên tách riêng các chức năng để bảo đảm hoạt động.

RAM
Metadata servers và monitors hoạt động liên tục để phục vụ cho dữ liệu, vì vậy cần số lượng RAM kha khá, thông thường 1Gb RAM cho mỗi instance daemon. OSD không cần nhiều RAM cho các hoạt động của nó, thông thường vào khoảng 500Mb cho mỗi instance daemon. Tuy nhiên, trong quá trình recovery sẽ cần nhiều RAM, thường 1Gb cho mỗi 1Tb lưu trữ trên mỗi daemon. Nhìn chung, càng nhiều RAM càng tốt, nhất là khi RAM là tài nguyên rẻ nhất hiện nay đối với máy tính, máy chủ.

Data Storage
Đây là yếu tố quan trọng nhất, cần cân nhắc thật kỹ giữa vấn đề chi phí và năng lực hệ thống.

OSDs có thể là SSD, HDD hoặc NVME sử dụng để lưu trữ data. Khuyến cáo sử dụng các đĩa cứng từ 1Tb trở lên. Cần quan tâm đến cách tính chi phí cho mỗi 1Gb lưu trữ. Ví dụ xài các đĩa cứng 1Tb ($75) thì chi phí ra sao ($0.07/1Gb) so với xài 1 đĩa cứng 3Tb ($150) thì chi phí như thế nào ($0.05/1Gb). Tuy nhiên, cần lưu ý khi dung lượng lưu trữ lớn, Ceph OSD Daemon cần thêm RAM, đặc biệt trong quá trình rebalancing, backfilling, và recovery. Khuyến cáo: 1Gb RAM cho 1 Tb không gian lưu trữ.

Các điểm quan trọng cần lưu ý khi triển khai đó là cần tách biệt các disk cho HĐH (OS) với các Ceph OSD Daemon, tránh việc chạy chung nhiều Ceph OSD daemon trên 1 disk, cũng như tránh chạy Ceph OSD trên MDS hay monitoring trên cùng 1 disk. Ngoài ra, OS, OSD Data, và OSD Journals nên được đặt trên các disk khác nhau, trong đó Journals sử dụng các đĩa cứng SSD để tăng tốc độ truy xuất dữ liệu cho các node lưu trữ OSD.

Việc thiết kế, sử dụng các đĩa cứng trong hệ thống CEPH thực tế như thế nào tùy thuộc vào nhu cầu của từng hệ thống.