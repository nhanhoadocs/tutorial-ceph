# Ceph Object Gateway (Ceph RADOSGW)

<p align="center">
<img src="../../images/ceph-radosgw.png">
</p>


Ceph Object Gateway, hay RADOS Gateway, là một proxy chuyển các request HTTP thành các RADOS request và ngược lại, cung cấp RESTful object storage, tương thích với S3 và Swift. Ceph Object Storage sử dụng Ceph Object Gateway Daemon (radosgw) để tương tác với librgw và Ceph cluster, librados. Nó sử dụng một module FastCGI  là libfcgi, và có thể sử dụng với bất cứ Server web tương thích với FASTCGI nào. Ceph Object Store hỗ trợ 3 giao diện sau:

- **S3**: Cung cấp Amazon S3 RESTful API.
- **Swift**: Cung cấp OpenStack Swift API. Ceph Object Gateway có thể thay thê Swift.
- **Admin**: Hỗ trợ quản trị Ceph Cluster thông qua HTTP RESTful API.

Ceph Object Gateway có phương thức quản lý người dùng riêng. Cả S3 và Swift API chia sẻ cùng một namespace chung trong Ceph cluster, do đó có thể ghi dữ liệu từ một API và lấy từ một API khác. Để xử lý nhanh, nó có thể dừng RAM làm cache metadata. Có thể dùng nhiều Gateway và sử dụng Load balancer để cân bằng tải vào Object Storage. Hiệu năng được cải thiện thông qua việc cắt nhỏ các REST object thành các RADOS object. Bên cạnh S3 và Swift API. Ứng dụng có thể bypass qua RADOS gateway và truy xuất trực tiếp tới librados, thường được sử dụng trong các ưgs dụng doanh nghiệp đòi hỏi hiệu năng cao. Ceph cho phép truy xuất trực tiếp tới cluster, điều khác biệt so với các hệ thống lưu trữ khác vốn giới hạn về các interface giao tiếp.