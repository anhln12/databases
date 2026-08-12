Bước 1: Export toàn bộ database từ Server cũ

```
mysqldump -u root -p --all-databases --routines --triggers --events --single-transaction | gzip > all_databases_$(date +%F).sql.gz
```

Lưu ý: Flag --single-transaction cực kỳ quan trọng để backup InnoDB an toàn mà không làm khóa bảng (lock table) trên môi trường production.

Bước 2: Cp file backup sang Server mới
```
scp all_databases_$(date +%F).sql.gz root@<IP_SERVER_MOI>:/opt/
```

Bước 3: Import vào Server mới
```
gunzip < /opt/all_databases_$(date +%F).sql.gz | mysql -u root -p
```

Bước 4: Flush Privileges (Đồng bộ quyền user)

Vì --all-databases bao gồm cả bảng mysql (chứa user và quyền), sau khi import xong cần chạy lệnh này để MySQL cập nhật lại bảng quyền:

```
mysql -u root -p -e "FLUSH PRIVILEGES;"
```
