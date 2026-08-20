Để đảm ảo hiệu năng cao, ổn định và chịu tải tốt:
1. Tối ưu Connection Pool (Giảm tải kết nối đến Mysql Backend)
ProxySQL hoạt động như một bộ đệm kết nối. Nếu không cấu hình khéo, MySQL backend sẽ dễ bị quá tải lượng kết nối đồng thời từ ứng dụng.
```
-- Tăng số kết nối tối đa mỗi backend server có thể nhận từ ProxySQL
UPDATE global_variables SET variable_value = '2000' WHERE variable_name = 'mysql-max_connections';

-- Thời gian giữ kết nối nhàn rỗi (timeout) để tái sử dụng, tránh việc đóng/mở liên tục
UPDATE global_variables SET variable_value = '120' WHERE variable_name = 'mysql-connection_max_age_ms';

-- Đẩy thay đổi xuống Runtime và Disk
LOAD MYSQL VARIABLES TO RUNTIME;
SAVE MYSQL VARIABLES TO DISK;
```
2. Bật Query Caching (Tăng tốc các câu lệnh SELECT lặp lại)
Nếu ứng dụng của bạn có nhiều câu truy vấn đọc ít thay đổi (như danh mục, cấu hình hệ thống), việc cache lại ở ProxySQL sẽ giúp giảm tải trực tiếp cho CPU của MySQL backend.
```
-- Bật tính năng query cache toàn cục
UPDATE global_variables SET variable_value = '1' WHERE variable_name = 'mysql-query_cache_size_MB'; -- Ví dụ cấp 256MB RAM cho cache

-- Cấu hình thời gian sống mặc định cho cache (tính bằng mili-giây, ví dụ 5 giây)
UPDATE global_variables SET variable_value = '5000' WHERE variable_name = 'mysql-query_processor_iterations';

LOAD MYSQL VARIABLES TO RUNTIME;
SAVE MYSQL VARIABLES TO DISK;
```

3. Tối ưu Thread Pool và Xử lý Lỗi (Multiplexing)
```
-- Đảm bảo cơ chế Multiplexing luôn được bật tối đa (mặc định là bật, kiểm tra lại)
SELECT variable_value FROM global_variables WHERE variable_name = 'mysql-multiplexing';

-- Tăng số lượng worker threads của ProxySQL tương ứng với số CPU Core của server
-- Ví dụ server có 4 cores:
UPDATE global_variables SET variable_value = '4' WHERE variable_name = 'admin-threads';
```

4. Giám sát & Bắt lỗi chậm (Query Rules Logging)
```
-- Bật log các câu truy vấn thực thi vượt ngưỡng thời gian (ví dụ: > 1000ms)
UPDATE global_variables SET variable_value = '1000' WHERE variable_name = 'mysql-log_slow_queries';

LOAD MYSQL VARIABLES TO RUNTIME;
SAVE MYSQL VARIABLES TO DISK;
```
