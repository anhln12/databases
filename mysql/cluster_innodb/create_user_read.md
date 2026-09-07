```
Application
    |
    v
 ProxySQL
    |
    +---- writer_hostgroup 10  ---> PRIMARY
    |
    +---- reader_hostgroup 20  ---> SECONDARY 1
    |                         ---> SECONDARY 2
    |                         ---> ...
```

1. Tạo user Read/Only trên MySQL Cluster

Lưu ý quan trọng: InnoDB Cluster dùng Group Replication, nên user phải được tạo trên PRIMARY và sẽ replicate sang các node còn lại.

Tạo user
```
CREATE USER 'app_read'@'%' IDENTIFIED BY 'MậtKhẩuMạnh';
GRANT SELECT ON *.* TO 'app_read'@'%';
SHOW GRANTS FOR 'app_read'@'%';
```

Nếu chỉ cho đọc một database, tốt hơn:
```
CREATE USER 'app_read'@'%' IDENTIFIED BY 'MậtKhẩuMạnh';
GRANT SELECT ON my_database.* TO 'app_read'@'%';
```

Kiểm tra lại quyền
```
SHOW GRANTS FOR 'app_read'@'%';

Không xuất hiện
INSERT
UPDATE
DELETE
CREATE
DROP
ALTER
TRUNCATE
```

2. Kiểm tra ProxySQL Hostgroup
```
SELECT hostgroup_id, hostname, port, status, weight
FROM runtime_mysql_servers
ORDER BY hostgroup_id, hostname;
```

3. Tạo user trên ProxySQL

Đăng nhập ProxySQL Admin
```
mysql -h 127.0.0.1 -P6032 -u admin -p
```

Thêm user:
```
INSERT INTO mysql_users
(
    username,
    password,
    default_hostgroup,
    transaction_persistent,
    active
)
VALUES
(
    'app_read',
    'MậtKhẩuMạnh',
    20,
    0,
    1
);
```

Load user mới
```
LOAD MYSQL USERS TO RUNTIME;
SAVE MYSQL USERS TO DISK;
```

4. Khuyến nghị có thể cấu hình chặt hơn

Thêm query rule
```
INSERT INTO mysql_query_rules
(
    rule_id,
    username,
    match_digest,
    destination_hostgroup,
    apply
)
VALUES
(
    100,
    'app_read',
    '^SELECT',
    20,
    1
);
```

Nếu muốn các truy vấn ko phải Select không được chạy, có thể thêm rule deny:
```
INSERT INTO mysql_query_rules
(
    rule_id,
    username,
    match_digest,
    error_msg,
    apply
)
VALUES
(
    101,
    'app_read',
    '^(INSERT|UPDATE|DELETE|REPLACE|ALTER|DROP|TRUNCATE|CREATE)',
    'READ ONLY USER',
    1
);
```

Sau đó:
```
LOAD MYSQL QUERY RULES TO RUNTIME;
SAVE MYSQL QUERY RULES TO DISK;
```

5. Kiểm tra từ application

Application kết nối
```
mysql -h <PROXYSQL_IP> -P6033 -u app_read -p
```

Kiểm tra:
```
SELECT @@hostname;
```

Thử select
```
SELECT COUNT(*) FROM my_database.my_table;
```

Thử create table:
```
CREATE TABLE test_readonly (
    id BIGINT NOT NULL AUTO_INCREMENT,
    username VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id)
);
``

Với user chỉ có SELECT, phải nhận lỗi tương tự:
```
ERROR 1142 (42000): CREATE command denied to user 'readonly_user'@'...'
```

6. Kiểm tra ProxySQL thực sự route vào Slave
```
SELECT hostgroup,
       srv_host,
       srv_port,
       Queries
FROM stats_mysql_connection_pool
WHERE hostgroup IN (10,20);
```





   
