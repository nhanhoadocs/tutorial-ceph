## Hướng dẫn cách tắt/bật cluster


# Tắt (không thực hiện bước tiếp, nếu các bước trước không thoả mãn).
1. Dừng sử dụng RBD images/Rados Gateway trên tất cả clients.
2. Hãy chắc chắn cluster đang ở trạng thái "healthy".
3. Bật cờ noout, norecover, norebalance, nobackfill, nodown and pause.

```sh
##Chạy trên ceph01 (mng node)

ceph osd set noout
ceph osd set norecover
ceph osd set norebalance
ceph osd set nobackfill
ceph osd set nodown
ceph osd set pause
```
> Status hiển thị
```
    OSDMAP_FLAGS: pauserd,pausewr,nodown,noout,nobackfill,norebalance,norecover flag(s) set 
```
4. Shutdown lần lượt osd node.
5. Shutdown lần lượt monitor node.
6. Shutdown admin node.


# Bật

1. Bật nguồn admin node.
2. Bật nguồn monitor node.
3. Bật nguồn osd node.
4. Đợi tới khi tất cả các node được bật, kiểm tra kết nối thành công giữa các node.
5. Tắt cờ noout,norecover,noreblance, nobackfill, nodown and pause.

```sh
##Run on ceph1(mng node)
ceph osd unset noout
ceph osd unset norecover
ceph osd unset norebalance
ceph osd unset nobackfill
ceph osd unset nodown
ceph osd unset pause
```

6. Kiểm tra trạng thái của cluster và kết nối lại từ các client.

# Links tham khảo:
https://ceph.io/planet/how-to-do-a-ceph-cluster-maintenance-shutdown/
