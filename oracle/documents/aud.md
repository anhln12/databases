Các file .aud này là Audit Trail files (file ghi log kiểm toán) của Oracle Database. Chúng được sinh ra do cấu hình bảo mật hoặc thông số khởi tạo của database nhằm ghi lại các hoạt động kết nối, câu lệnh, hoặc sự kiện hệ thống.

Traditional Auditing (Kiểm toán truyền thống): Mặc định, khi tính năng audit của Oracle được bật hoặc một số sự kiện hệ thống cốt lõi xảy ra (như DBA đăng nhập, lệnh quản trị, v.v.), Oracle sẽ tự động ghi lại các bản ghi này ra các file .aud thay vì lưu trong bảng (table) của database để tránh làm nghẽn hệ thống khi DB chưa sẵn sàng

Tên file cấu trúc: [sid]_ora_[pid]_[timestamp].aud (ví dụ bss2_ora_12274_...aud nghĩa là instance SID là bss2, process ID tương ứng).

Các Policy / Tham số quyết định việc sinh file này
```
-- 1. Kiểm tra thư mục chứa file audit (AUDIT_FILE_DEST)
SHOW PARAMETER audit_file_dest;

-- 2. Kiểm tra loại audit đang bật (OS, DB, hoặc NONE)
SHOW PARAMETER audit_trail;
```

AUDIT_TRAIL:
- Nếu là NONE: Vẫn có thể sinh các file audit tối thiểu (như OS authentication, kết nối bằng AS SYSDBA).
- Nếu là DB hoặc DB, EXTENDED: Một số log chuyển vào bảng DBA_AUDIT_TRAIL, nhưng các kết nối đặc quyền (SYS) vẫn ghi ra file .aud ở OS.
- Nếu là OS: Toàn bộ log audit được đẩy ra file .aud.

Giải pháp tự động hóa

Vì không thể xóa qua SQL, bạn hãy đặt cronjob chạy hàng ngày dưới user oracle để tự động dọn dẹp các file cũ hơn 30 ngày:

```
0 2 * * * find /u01/app/oracle/database/19.3.0/admin/bss/adump -name "*.aud" -mtime +30 -delete
```



