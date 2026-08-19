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
