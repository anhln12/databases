1. Kiểm tra tiến trình MRP trên Standby
```
SELECT PROCESS, STATUS, THREAD#, SEQUENCE# FROM V$MANAGED_STANDBY;
```

```
SQL> SELECT PROCESS, STATUS, THREAD#, SEQUENCE# FROM V$MANAGED_STANDBY;
PROCESS   STATUS          THREAD#  SEQUENCE#
--------- ------------ ---------- ----------
ARCH      CONNECTED             0          0
DGRD      ALLOCATED             0          0
DGRD      ALLOCATED             0          0
ARCH      CONNECTED             0          0
ARCH      CONNECTED             0          0
ARCH      CONNECTED             0          0
ARCH      CONNECTED             0          0
MRP0      WAIT_FOR_GAP          1      14363
RFS       IDLE                  0          0
RFS       IDLE                  0          0
10 rows selected. 
```

Nguyên nhân: Tiến trình MRP0 đang ở trạng thái WAIT_FOR_GAP tại sequence 14363, có nghĩa là Standby đang thiếu archived log sequence 14363 (do bị đứt quãng truyền dữ liệu từ Primary sang), dẫn đến sequence 14364 (APPLIED = NO) phía sau chưa thể áp dụng.

2.  Kiểm tra file logs số 14363
```
-rw-r-----. 1 oracle oinstall     1024 Jul 15 03:06 o1_mf_1_14363_o5f5p76d_.arc 
```

File 14363 thực sự tồn tại trên Primary (dung lượng quá nhỏ so với mức 5MB+)

Giải pháp: Khi không còn bản backup và file 14363 trên Primary bị hỏng/rỗng (1KB), cách duy nhất để cứu Standby mà không phải dựng lại từ đầu (Rebuild) là bỏ qua sequence bị lỗi hoặc tăng Current Log Sequence bằng cách thực hiện một bản Incremental Backup từ Primary sang Standby.

Các bước thực hiện Incremental Backup để vá GAP

Bước 1: Trên Standby (Hủy MRP) và chuyển DB về chế đô mount (Chỉ khi DB ở chế độ mount mới thực hiện được việc restore)
```
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE CANCEL;
SELECT CURRENT_SCN FROM V$DATABASE; => Lấy CHECKPOINT_CHANGE# hiện tại của Standby
# Đóng và mount lại database trên Standby để giải phóng toàn bộ lock trên datafile
SQL> SHUTDOWN IMMEDIATE;
SQL> STARTUP MOUNT;
```

Bước 2: Trên Primary thực hiện tạo incremental backup từ SCN của Standby
```
RMAN> BACKUP INCREMENTAL FROM SCN 331303451 DATABASE FORMAT '/tmp/incr_stby_%U.bck';
```

Bước 3: Copy toàn bộ Backup từ /tmp trên Primary sang thư mục tương tứng trên Standby

Bước 4: Trên Stadby tiến hành catalog và recover
```
RMAN> CATALOG START WITH '/tmp/';
RMAN> RECOVER DATABASE;
```

```
RMAN> CATALOG START WITH '/tmp/';

using target database control file instead of recovery catalog
searching for all files that match the pattern /tmp/

List of Files Unknown to the Database
=====================================
File Name: /tmp/incr_stby_v24v5j10_13282_1_1.bck
File Name: /tmp/incr_stby_v34v5j10_13283_1_1.bck
File Name: /tmp/incr_stby_v44v5j10_13284_1_1.bck
File Name: /tmp/incr_stby_v44v5j10_13284_2_1.bck
File Name: /tmp/incr_stby_v44v5j10_13284_3_1.bck
File Name: /tmp/incr_stby_v44v5j10_13284_4_1.bck
File Name: /tmp/incr_stby_v44v5j10_13284_5_1.bck
File Name: /tmp/incr_stby_v44v5j10_13284_6_1.bck
File Name: /tmp/incr_stby_v44v5j10_13284_7_1.bck
File Name: /tmp/incr_stby_v54v5j10_13285_1_1.bck
File Name: /tmp/incr_stby_v64v5j5l_13286_1_1.bck
File Name: /tmp/incr_stby_v74v5j8l_13287_1_1.bck
File Name: /tmp/incr_stby_v84v5j9f_13288_1_1.bck
File Name: /tmp/incr_stby_vc4v5j9r_13292_1_1.bck
File Name: /tmp/incr_stby_vd4v5j9v_13293_1_1.bck

Do you really want to catalog the above files (enter YES or NO)? YES
cataloging files...
cataloging done

List of Cataloged Files
=======================
File Name: /tmp/incr_stby_v24v5j10_13282_1_1.bck
File Name: /tmp/incr_stby_v34v5j10_13283_1_1.bck
File Name: /tmp/incr_stby_v44v5j10_13284_1_1.bck
File Name: /tmp/incr_stby_v44v5j10_13284_2_1.bck
File Name: /tmp/incr_stby_v44v5j10_13284_3_1.bck
File Name: /tmp/incr_stby_v44v5j10_13284_4_1.bck
File Name: /tmp/incr_stby_v44v5j10_13284_5_1.bck
File Name: /tmp/incr_stby_v44v5j10_13284_6_1.bck
File Name: /tmp/incr_stby_v44v5j10_13284_7_1.bck
File Name: /tmp/incr_stby_v54v5j10_13285_1_1.bck
File Name: /tmp/incr_stby_v64v5j5l_13286_1_1.bck
File Name: /tmp/incr_stby_v74v5j8l_13287_1_1.bck
File Name: /tmp/incr_stby_v84v5j9f_13288_1_1.bck
File Name: /tmp/incr_stby_vc4v5j9r_13292_1_1.bck
File Name: /tmp/incr_stby_vd4v5j9v_13293_1_1.bck

RMAN> RECOVER DATABASE;
Starting recover at 07-AUG-26
using target database control file instead of recovery catalog
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=745 device type=DISK
allocated channel: ORA_DISK_2
channel ORA_DISK_2: SID=5 device type=DISK
allocated channel: ORA_DISK_3
channel ORA_DISK_3: SID=129 device type=DISK
allocated channel: ORA_DISK_4
channel ORA_DISK_4: SID=254 device type=DISK
channel ORA_DISK_1: starting incremental datafile backup set restore
channel ORA_DISK_1: specifying datafile(s) to restore from backup set
destination for restore of datafile 00001: /u02/oradata/ORADBR/system01.dbf
channel ORA_DISK_1: reading from backup piece /tmp/incr_stby_v74v5j8l_13287_1_1.bck
channel ORA_DISK_2: starting incremental datafile backup set restore
channel ORA_DISK_2: specifying datafile(s) to restore from backup set
destination for restore of datafile 00003: /u02/oradata/ORADBR/sysaux01.dbf
channel ORA_DISK_2: reading from backup piece /tmp/incr_stby_v44v5j10_13284_1_1.bck
channel ORA_DISK_3: starting incremental datafile backup set restore
channel ORA_DISK_3: specifying datafile(s) to restore from backup set
destination for restore of datafile 00004: /u02/oradata/ORADBR/undotbs01.dbf
channel ORA_DISK_3: reading from backup piece /tmp/incr_stby_v84v5j9f_13288_1_1.bck
channel ORA_DISK_4: starting incremental datafile backup set restore
channel ORA_DISK_4: specifying datafile(s) to restore from backup set
destination for restore of datafile 00007: /u02/oradata/ORADBR/users01.dbf
channel ORA_DISK_4: reading from backup piece /tmp/incr_stby_vd4v5j9v_13293_1_1.bck
channel ORA_DISK_4: piece handle=/tmp/incr_stby_vd4v5j9v_13293_1_1.bck tag=TAG20260807T083856
channel ORA_DISK_4: restored backup piece 1
channel ORA_DISK_4: restore complete, elapsed time: 00:00:01
channel ORA_DISK_4: starting incremental datafile backup set restore
channel ORA_DISK_4: specifying datafile(s) to restore from backup set
destination for restore of datafile 00009: /u02/oradata/ORADBR/pdb1/system01.dbf
destination for restore of datafile 00010: /u02/oradata/ORADBR/pdb1/sysaux01.dbf
destination for restore of datafile 00012: /u02/oradata/ORADBR/pdb1/users01.dbf
channel ORA_DISK_4: reading from backup piece /tmp/incr_stby_v24v5j10_13282_1_1.bck
channel ORA_DISK_1: piece handle=/tmp/incr_stby_v74v5j8l_13287_1_1.bck tag=TAG20260807T083856
channel ORA_DISK_1: restored backup piece 1
channel ORA_DISK_1: restore complete, elapsed time: 00:00:08
channel ORA_DISK_1: starting incremental datafile backup set restore
channel ORA_DISK_1: specifying datafile(s) to restore from backup set
destination for restore of datafile 00011: /u02/oradata/ORADBR/pdb1/undotbs01.dbf
destination for restore of datafile 00013: /u02/oradata/ORADBR/pdb1/api_gateway_01.dbf
destination for restore of datafile 00015: /u02/oradata/ORADBR/pdb1/api_gateway_03.dbf
destination for restore of datafile 00018: /u02/oradata/ORADBR/pdb1/users02.dbf
channel ORA_DISK_1: reading from backup piece /tmp/incr_stby_v64v5j5l_13286_1_1.bck
channel ORA_DISK_3: piece handle=/tmp/incr_stby_v84v5j9f_13288_1_1.bck tag=TAG20260807T083856
channel ORA_DISK_3: restored backup piece 1
channel ORA_DISK_3: restore complete, elapsed time: 00:00:11
channel ORA_DISK_3: starting incremental datafile backup set restore
channel ORA_DISK_3: specifying datafile(s) to restore from backup set
destination for restore of datafile 00014: /u02/oradata/ORADBR/pdb1/api_gateway_02.dbf
destination for restore of datafile 00016: /u02/oradata/ORADBR/pdb1/api_gateway_04.dbf
destination for restore of datafile 00019: /u02/oradata/ORADBR/pdb1/users03.dbf
destination for restore of datafile 00023: /u02/oradata/ORADBR/pdb1/sync_info_c06_02.dbf
channel ORA_DISK_3: reading from backup piece /tmp/incr_stby_v54v5j10_13285_1_1.bck
channel ORA_DISK_4: piece handle=/tmp/incr_stby_v24v5j10_13282_1_1.bck tag=TAG20260807T083856
channel ORA_DISK_4: restored backup piece 1
channel ORA_DISK_4: restore complete, elapsed time: 00:00:17
channel ORA_DISK_4: starting incremental datafile backup set restore
channel ORA_DISK_4: specifying datafile(s) to restore from backup set
destination for restore of datafile 00017: /u02/oradata/ORADBR/pdb1/avdf_aud_data01.dbf
destination for restore of datafile 00020: /u02/oradata/ORADBR/pdb1/users04.dbf
destination for restore of datafile 00021: /u02/oradata/ORADBR/pdb1/sync_info_c06_01.dbf
destination for restore of datafile 00022: /u02/oradata/ORADBR/pdb1/callback_c06_01.dbf
channel ORA_DISK_4: reading from backup piece /tmp/incr_stby_v34v5j10_13283_1_1.bck
channel ORA_DISK_1: piece handle=/tmp/incr_stby_v64v5j5l_13286_1_1.bck tag=TAG20260807T083856
channel ORA_DISK_1: restored backup piece 1
channel ORA_DISK_1: restore complete, elapsed time: 00:00:25
channel ORA_DISK_3: piece handle=/tmp/incr_stby_v54v5j10_13285_1_1.bck tag=TAG20260807T083856
channel ORA_DISK_3: restored backup piece 1
channel ORA_DISK_3: restore complete, elapsed time: 00:00:22
channel ORA_DISK_4: piece handle=/tmp/incr_stby_v34v5j10_13283_1_1.bck tag=TAG20260807T083856
channel ORA_DISK_4: restored backup piece 1
channel ORA_DISK_4: restore complete, elapsed time: 00:00:55
channel ORA_DISK_2: piece handle=/tmp/incr_stby_v44v5j10_13284_1_1.bck tag=TAG20260807T083856
channel ORA_DISK_2: restored backup piece 1
channel ORA_DISK_2: reading from backup piece /tmp/incr_stby_v44v5j10_13284_2_1.bck
channel ORA_DISK_2: piece handle=/tmp/incr_stby_v44v5j10_13284_2_1.bck tag=TAG20260807T083856
channel ORA_DISK_2: restored backup piece 2
channel ORA_DISK_2: reading from backup piece /tmp/incr_stby_v44v5j10_13284_3_1.bck
channel ORA_DISK_2: piece handle=/tmp/incr_stby_v44v5j10_13284_3_1.bck tag=TAG20260807T083856
channel ORA_DISK_2: restored backup piece 3
channel ORA_DISK_2: reading from backup piece /tmp/incr_stby_v44v5j10_13284_4_1.bck
channel ORA_DISK_2: piece handle=/tmp/incr_stby_v44v5j10_13284_4_1.bck tag=TAG20260807T083856
channel ORA_DISK_2: restored backup piece 4
channel ORA_DISK_2: reading from backup piece /tmp/incr_stby_v44v5j10_13284_5_1.bck
channel ORA_DISK_2: piece handle=/tmp/incr_stby_v44v5j10_13284_5_1.bck tag=TAG20260807T083856
channel ORA_DISK_2: restored backup piece 5
channel ORA_DISK_2: reading from backup piece /tmp/incr_stby_v44v5j10_13284_6_1.bck
channel ORA_DISK_2: piece handle=/tmp/incr_stby_v44v5j10_13284_6_1.bck tag=TAG20260807T083856
channel ORA_DISK_2: restored backup piece 6
channel ORA_DISK_2: reading from backup piece /tmp/incr_stby_v44v5j10_13284_7_1.bck
channel ORA_DISK_2: piece handle=/tmp/incr_stby_v44v5j10_13284_7_1.bck tag=TAG20260807T083856
channel ORA_DISK_2: restored backup piece 7
channel ORA_DISK_2: restore complete, elapsed time: 00:06:13

starting media recovery

archived log for thread 1 with sequence 14364 is already on disk as file /u02/archive/ORADBR/archivelog/2026_07_15/o1_mf_1_14364_o5f5p790_.arc
media recovery complete, elapsed time: 00:00:00
Finished recover at 07-AUG-26
```

Trạng thái: Quá trình Recovery bằng Incremental Backup đã hoàn tất thành công (media recovery complete). Standby database đã được đồng bộ vượt qua được khoảng gap bị lỗi trước đó.

Bước 5: Khởi động lại tiến trình đồng bộ (MRP) trên Standby





