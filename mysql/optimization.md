```
/etc/mysql/
├── my.cnf
├── conf.d/
│   └── mysql.cnf
└── mysql.conf.d/
    └── mysqld.cnf
```

Trong đó:
- /etc/mysql/my.cnf → chỉ include, không cần sửa.
- /etc/mysql/conf.d/mysql.cnf → cấu hình client.
- /etc/mysql/mysql.conf.d/mysqld.cnf → cấu hình MySQL Server.

Tôi khuyên tạo thêm /etc/mysql/mysql.conf.d/99-production-tuning.cnf → nơi đặt tuning riêng của bạn.

Bước tiếp theo:

Lấy thông tin hardware
```
free -h
echo "===== CPU ====="
lscpu | egrep 'CPU\(s\)|Core|Socket|Thread'
echo "===== DISK ====="
lsblk -o NAME,SIZE,TYPE,ROTA,MODEL
echo "===== FILESYSTEM ====="
df -hT /var/lib/mysql
```

Lấy workload MySQL
```
SHOW GLOBAL STATUS LIKE 'Threads_connected';
SHOW GLOBAL STATUS LIKE 'Threads_running';
SHOW GLOBAL STATUS LIKE 'Max_used_connections';
SHOW GLOBAL STATUS LIKE 'Questions';
SHOW GLOBAL STATUS LIKE 'Queries';
SHOW GLOBAL STATUS LIKE 'Connections';
SHOW GLOBAL STATUS LIKE 'Created_tmp%';
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool%';
```

```
SHOW VARIABLES WHERE Variable_name IN (
    'max_connections',
    'innodb_buffer_pool_size',
    'innodb_buffer_pool_instances',
    'innodb_log_buffer_size',
    'innodb_log_file_size',
    'innodb_log_files_in_group',
    'innodb_flush_log_at_trx_commit',
    'innodb_flush_method',
    'sync_binlog',
    'binlog_format',
    'binlog_expire_logs_seconds',
    'table_open_cache',
    'table_definition_cache',
    'thread_cache_size',
    'tmp_table_size',
    'max_heap_table_size'
);
```

Tôi sẽ tạo file
```
/etc/mysql/mysql.conf.d/99-production-tuning.cnf
```

Với các nhóm
```
MySQL
├── Memory / InnoDB Buffer Pool
├── Connection
├── Thread / Table Cache
├── Redo / Flush
├── Binary Log
├── Group Replication
├── Slow Query Log
└── Performance / Temporary Tables
```


Config mặc định
|Parameter|hiện tại|Đánh giá|
|---|---|---|
|innodb_buffer_pool_size|128 MB||
|max_connections|1500|Quá cao so với Ram 8GB|
|innodb_buffer_pool_instances|1|✅ Có thể giữ|
|innodb_log_file_size|48 MB|🟠 Khá thấp|
|innodb_log_buffer_size|16 MB|🟡 Có thể tăng|
|table_open_cache|4000|✅|
|table_definition_cache|2000|✅|
|thread_cache_size|23|🟡 Có thể tăng|
|tmp_table_size|16 MB|✅|
|max_heap_table_size|16 MB|✅|
|sync_binlog|1|✅ Giữ|
|innodb_flush_log_at_trx_commit|1|✅ Giữ|
|binlog_format|OW|✅ Giữ|
|binlog_expire_logs_seconds|30 ngày|

```
/etc/mysql/mysql.conf.d/99-production-tuning.cnf

[mysqld]

# =========================================================
# CONNECTION
# =========================================================

# ProxySQL is in front of MySQL, so don't allow excessive
# backend connections on an 8 GB RAM server.
max_connections = 3000

thread_cache_size = 50


# =========================================================
# INNODB BUFFER POOL
# =========================================================

# Server has ~8 GB RAM.
# Start with 4 GB for InnoDB and leave enough RAM for
# MySQL per-connection buffers, Group Replication and OS.
innodb_buffer_pool_size = 4G

innodb_buffer_pool_instances = 1


# =========================================================
# INNODB LOG BUFFER
# =========================================================

innodb_log_buffer_size = 64M


# =========================================================
# INNODB DURABILITY
# =========================================================

# Keep maximum durability for production / replication.
innodb_flush_log_at_trx_commit = 1


# =========================================================
# TABLE CACHE
# =========================================================

table_open_cache = 4000
table_definition_cache = 2000


# =========================================================
# TEMPORARY TABLE
# =========================================================

tmp_table_size = 16M
max_heap_table_size = 16M


# =========================================================
# BINARY LOG
# =========================================================

# Required for Group Replication.
binlog_format = ROW

# Maximum durability for binary log.
sync_binlog = 1

# Keep binary logs for 7 days.
binlog_expire_logs_seconds = 604800


# =========================================================
# SLOW QUERY LOG
# =========================================================

# Enable slow query logging.
slow_query_log = ON

slow_query_log_file = /var/log/mysql/mysql-slow.log

# Log queries taking more than 5 seconds.
long_query_time = 5

# Do not log every query that doesn't use an index.
# This avoids excessive log volume.
log_queries_not_using_indexes = OFF

min_examined_row_limit = 0
```






