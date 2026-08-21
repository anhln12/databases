Mục tiêu

```
ProxySQL
   ↓
log
   ↓
100 MB
   ↓
rotate
   ↓
giữ 7 file
   ↓
xóa file cũ
```

Tạo file
```
vi /etc/logrotate.d/proxysql
```

