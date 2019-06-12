# Ceph PG status

Toàn bộ data của Ceph được lưu trữ dưới dạng Object và được phân bố trong các Placement Group hay còn gọi là PG. Bình thường CRUSH sẽ tự động tính toán việc lưu trữ data, khôi phục dữ liệu, nhân bản dữ liệu ... Hệ thống sẽ dựa trên các `map` của từng service để kiểm tra trạng thái thay đổi của cụm Ceph, Service monitor sẽ đảm nhiệm vấn đề này. 

Ceph sử dụng một khái niệm gọi là PG status để cho người dùng biết được là trạng thái dữ liệu trên cụm đang như thế nào đối với từng PG 

Thao tác để kiểm tra thường là ceph -s hoặc ceph -w hoặc có thể là trên Dashboad. Trạng thái tối ưu và hoàn hảo nhất cho toàn bộ PG là `active+clean` dưới đây là 1 ví dụ 

```sh 
  cluster:
    id:     3f59d96e-2725-4b0c-b726-baae012b3928
    health: HEALTH_OK
 
  services:
    mon: 1 daemons, quorum cephaio (age 30h)
    mgr: cephaio(active, since 30h)
    osd: 3 osds: 3 up (since 37h), 3 in (since 7d)
 
  data:
    pools:   1 pools, 512 pgs
    objects: 116 objects, 137 MiB
    usage:   3.3 GiB used, 144 GiB / 147 GiB avail
    pgs:     512 active+clean

```

Chúng ta có thể thấy trạng thái cụm Ceph là HEALTH_OK và toàn bộ PG của cụm là 512 đều ở trạng thái là `active+clean`

Ngoài ra Ceph còn có các trạng thái khác như sau 

|Trạng thái|Thông tin chi tiết                             |FIELD3                                                       |
|----------|-----------------------------------------------|-------------------------------------------------------------|
|`creating`| PG đang trong trạng thái khởi tạo             |                                                             |
|`activating`| PG khởi tạo xong đang trong trạng thái active hoăc chờ sử dụng|                                                             |
|`active`  | PG active sẵn sàng xử lý các request của client đọc ghi cập nhật ... |                                                             |
|`clean`   | Các bản ghi replicate đúng vị trí và đủ số lượng|                                                             |
|`down`    | Một bản sao có thiết bị hỏng và PG không sẵn sàng xử lý các request|                                                             |
|`scrubbing`| Ceph kiểm tra metadata của PG không nhất quán giữa các bản replica|                                                             |
|`deep`    | Ceph kiểm tra data trên các PG bằng cách kiểm tra checksums data |                                                             |
|`degraded`| Chưa replica đủ số bản ghi (x2 x3) theo cấu hình |                                                             |
|`inconsistent`| Kiểm tra sự không nhất quán của dữ liệu (sai vị trí thiếu bản ghi hoặc bản ghi không nhất quán)|                                                             |
|`peering` | PG đang trong quá trình chuẩn bị xử lý        |                                                             |
|`repair`  | Kiểm tra và sửa các PG có trạng thái `inconsistent` nếu nó thấy |                                                             |
|`recovering`| Hệ thống đang di chuyển đồng bộ các object đúng đủ bản sao của nó |                                                             |
|`forced_recovery`| Ưu tiên phục hồi PG bởi người dùng            |                                                             |
|`recovery_wait`| PG đang trong hàng đợi chờ để `recover`       |                                                             |
|`recovery_toofull`| PG đang chờ để recover nhưng đã vượt qua ngưỡng lưu trữ |                                                             |
|`backfilling`| Ceph đang quét và đồng bộ hóa toàn bộ nội dung của một PG thay vì suy ra nội dung nào cần được đồng bộ hóa từ nhật ký của các hoạt động gần đây. `backfilling` là một trường hợp đặc biệt của `recover`.|                                                             |
|`forced_backfill`| Ưu tiên backfill PG bởi người dùng            |                                                             |
|`backfill_wait`| PG đang trong hàng đợi chờ `backfill`         |                                                             |
|`backfill_toofull`|  PG đang trong hàng đợi chờ `backfill` nhưng đã vượt quá dung lượng lưu trữ của cụm|                                                             |
|`backfill_unfound`| PG dừng quá trình backfill vì không đối chiếu kiểm tra được dữ liệu của PG|                                                             |
|`incomplete`| Hệ thống kiểm tra thấy PG thiếu thông tin về dữ liệu của nó hoặc không có bản sao nào chắc chắn đúng. Cần thử start lại OSD đã hỏng nào có data đúng hoặc giảm `min_size` để cho phép khôi phục|                                                             |
|`stale`   | Trạng thái không xác định có thể chưa nhận được cập nhật từ lần thay đổi pg_map lần cuối cùng |                                                             |
|`remapped`| PG được lưu trữ ánh xạ qua 1 vị trí trên OSD khác theo sự chỉ định của CRUSH|
|`undersized`| Có ít bản sao hơn so với cấu hình của pool    |                                                             |
|`peered`  | Đã chuẩn bị xử lý nhưng chưa phục vụ các thao tác IO của client do ko đủ bản sao theo min_size |                                                             |
|`snaptrim`| Dọn dẹp bản snapshot                          |                                                             |
|`snaptrim_wait`| PG vào hàng đợi chờ dọn dẹp snapshot          |                                                             |
|`snaptrim_error`| Lỗi trong quá trình dọn dẹp snapshot          |                                                             |
