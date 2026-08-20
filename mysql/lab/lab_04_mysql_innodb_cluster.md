LAB 04: TRIỂN KHAI MYSQL INNODB CLUSTER (HIGH AVAILABILITY)

Mục tiêu:
1. Hiểu và triển khai được mô hình High Availability (HA) để đảm bảo tính sẵn sàng của dữ liệu
2. Sử dụng MySQL Shell để quản trị cụm máy chủ (Cluster)
3. Thực hiện kịch bản Failover: Giả lập Master node có vấn đề và quan sát hệ thống tự bầu chọn Master mới

PHẦN 1: CHUẨN BỊ MÔI TRƯỜNG (PREPARATION)

Yêu cầu: 03 Máy chủ Ubuntu Server 24.04 (hoặc 22.04)
1.1 Quy hoạch địa chỉ IP
|Tên máy|Vai trò|IP Address|
|---|---|---|
|VM01| Primary(Master)|192.168.1.10|
|VM02| Secondary(Slave)|192.168.1.20|
|VM03| Secondary(Salve)|192.168.1.30|

1.2 Cấu hình phân giải tên miền (Host Mapping)

Việc này giúp các máy gọi nhau bằng tên thay vì nhớ IP

Thực hiện trên cả 3 máy (VM01, VM02, VM03):

```
/etc/hosts
192.168.1.10 server10-prod
192.168.1.20 server11-prod
192.168.1.30 server12-prod
```

1.3 Cài dặt Mysql và Mysql Shell

```
# Update system packages
sudo apt update && sudo apt upgrade -y

# Download MySQL APT repository
wget https://dev.mysql.com/get/mysql-apt-config_0.8.39-1_all.deb
sudo dpkg -i mysql-apt-config_0.8.29-1_all.deb

# Select MySQL 8.4 LTS during configuration
sudo apt update
sudo apt install mysql-server mysql-shell -y

# Secure MySQL installation
sudo mysql_secure_installation

# Check service mysql
systemctl status mysql
```

PHẦN 2: CẤU HÌNH NGƯỜI DÙNG & BẢO MẬT

Cluster cần một tài khoản quản trị có quyền cao nhất để tự dồng bộ dữ liệu giữa các Node

Thực hiện trên cả 3 node

2.1 Tạo tài khoản Admin Cluster
```
mysql -u root -p
-- Tạo user quản trị
CREATE USER 'clusteradmin'@'%' IDENTIFIED WITH caching_sha2_password BY 'xxxxx';
-- Cấp toàn quyền
GRANT ALL PRIVILEGES ON *.* TO 'clusteradmin' WITH GRANT OPTION;
-- Áp dụng quyền
flush privileges;
```

2.2 Mở kết nối mạng (Bind Address)

Mặc định MySQL chỉ nghe ở 127.0.0.1. Cần mở để các node nhìn thấy nhau
```
bind-address = 0.0.0.0
```

2.3 Config Firewall

Nếu Ubuntu đang bật UFW, cần mở cổng 3306 và 33061 (Cổng giao tiếp nội bộ CLuster)
```
ufw allow 3306/tcp
ufw allow 33061/tcp
```

PHẦN 3: THIẾT LẬP CLUSTER (Sử dụng MYSQL SHELL)

3.1 Kiểm tra và config Instance
MySQL Shell có tính năng tự động phát hiện lỗi cấu hình và sửa chúng (như server-id gtid_mode ...)

Thực hiện lần lượt trên từng máy
```
mysqlsh
shell.connect('clusteradmin@server10-prod:3306');
dba.checkInstanceConfiguration('clusteradmin@server10-prod:3306');
dba.configureInstance('clusteradmin@server10-prod:3306');
```

Hệ thống sẽ hỏi:
```
Do you want to perform the required configuration changes? [y/n]: y
Do you want to restart the instance after configuring it? [y/n]: y
```

3.2 Khởi tạo Cluster

Chúng ta sẽ biến VM01 thành máy chủ đầu tiên (Primary) của cụm

1. Tại VM01 vào mysqlsh kết nối lại

```
mysqlsh
\connect clusteradmin@server10-prod:3306
var cluster = dba.createCluster('mycluster')
cluster.status()
```

Bạn sẽ thấy chỉ có VM01 trong danh sách topology

3.3 Thêm các Node vào Cluster

Chúng ta sẽ thêm VM02 và VM03 vào cụm
```
cluster.addInstance('clusteradmin@server11-prod:3306', {recoveryMethod: 'clone'});
cluster.addInstance('clusteradmin@server12-prod:3306', {recoveryMethod: 'clone'});
```

PHẦN 4: KIỂM TRA VÀ GIẢ LẬP SỤ CỐ (FAILOVER TEST)

Đây là phần quan trọng nhất để chứng minh hệ thống High Availability hoạt động

4.1 Kiểm tra trạng thái cluster

Yêu cầu quan sát: Cả 3 máy VM01, 02, 03 đều có trạng thái Online
Một máy có Role RW (Read/Writer - Primary) - Thường là VM01
Hai máy còn lại là RO (Read Only - Secondary)

4.2 Giả lập sự cố (Kill Master Node)
Tắt nguồn hoặc stop service mysql

4.3 Quan sát quá trình bàu chọn lại (Failover)
Kết quả mong đợi:
- VM01 sẽ có trạng thái Offline
- Hệ thống tự động bầu chọn VM02 or VM03 lên làm Primary

4.4. Khôi phục Node hỏng (Rejoin)
```
cluster.rejoinInstance('clusteradmin@server10-prod:3306')
```

Câu hỏi lý thuyết: Tạo sao trong môi trường Cluster, ta cần ít nhất 3 máy (số lẻ) thay vì 2 máy (Tìm hiểu về Split-brain và Quorum)











