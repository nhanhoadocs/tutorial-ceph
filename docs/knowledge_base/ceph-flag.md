## Ceph Flag Cluster
Thường việc cập nhật trạng thái cụm Ceph được thực hiện tự động và theo setting sẵn có. Nếu chúng ta cần thực hiện một vài thay đổi thì việc set flag (set cờ) cho cụm Ceph sẽ giúp chúng ta tăng tính chủ động trong việc quản trị cụm Ceph

Thao tác apply cho toàn Cluster
```
ceph osd set <flag>
ceph osd unset <flag>
```

- full - Set flag full chúng ta sẽ không thể ghi dữ liệu mới vào được cluster
- pauserd, pausewr - Tạm dừng thao tác read hoặc write trên cụm 
- noup - Khi set flag sẽ tạm ngưng thông báo khi có OSD up trong hệ thống
- nodown - Khi set flag sẽ tạm ngưng thông báo khi có OSD down trong hệ thống
- noin - Đối với các OSD đã down trước đó thì việc set flag này sẽ không tự động set OSD in và không cho phép OSD join vào CRUSHMAP khi OSD này start
- noout - Khi OSD down hệ thống sẽ không tự động set OSD out ra khỏi cluster (Thường là 10m) cho đến khi chúng ta gỡ flag
- nobackfill, norecover, norebalance - Suppend thao tác recovery và data rebalancing
- noscrub, nodeep_scrub - Disable scrubbing
- notieragent - Suppend cache tiering đang hoạt động


Thao tác set chỉ định OSD
```sh
ceph osd add-<flag> <osd-id>
ceph osd rm-<flag> <osd-id>
```

Đối với OSD chỉ định chỉ apply được các tham số 

- noup
- nodown
- noin
- noout

## Các ngữ cảnh sử dụng Flag
- Down node Ceph bảo trì
- Không cho phép thực hiện 1 task nào đó (scrub) trên cụm Ceph
- ...
