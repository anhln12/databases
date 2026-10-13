# Case study: Database PostgreSQL phình 159 GB vì Large Object mồ côi

*Bài chia sẻ nội bộ, rút kinh nghiệm cho Dev và DBA*

## 1. Tóm tắt

DB `scoredb` tăng lên **159 GB** và làm đầy ổ đĩa. Lần chạy `VACUUM` của quản trị thất bại vì không còn chỗ trống. Khi phân tích, phần lớn dung lượng **không nằm trong bảng nghiệp vụ** mà nằm ở các bảng catalog hệ thống:

| Đối tượng | Dung lượng |
| --- | --- |
| `pg_largeobject` | 63 GB |
| `pg_shdepend` | 32 GB |
| `pg_largeobject_metadata` | 15 GB |
| `tbl_log_request_score` | 40 GB (data 17 GB, index 23 GB) |
| `tbl_log_response_score` | 40 GB (data 17 GB, index 23 GB) |

Hơn 110 GB nằm trong catalog, một thứ mà hầu hết DBA hiếm khi phải quan tâm.

## 2. Nguyên nhân gốc

Bảng `tbl_leadgen_customers_to_partner_log` ghi lại JSON request và response gửi sang đối tác. Hai cột `json_request` và `json_response` có kiểu **`oid`**, nghĩa là app đang lưu JSON bằng **Large Object (LO)** chứ không lưu trực tiếp trong cột.

Chuỗi sự kiện dẫn tới sự cố:

1. Mỗi lần ghi log, app tạo **2 large object** (request và response).
2. Bảng chỉ giữ số OID, nội dung thật nằm trong `pg_largeobject`.
3. Khi dòng log bị xóa, **không ai `lo_unlink`** các object tương ứng.
4. Large object mồ côi vẫn là dữ liệu "còn sống" với Postgres nên `VACUUM` không dọn.
5. Mỗi LO còn sinh thêm dòng trong `pg_largeobject_metadata` và `pg_shdepend`, nên các catalog này phình theo.
6. Bảng nghiệp vụ nhìn rất nhỏ, nên không ai nghĩ tới việc kiểm tra chỗ này.

## 3. Large Object là gì, và vì sao nguy hiểm

Hình dung đơn giản: lưu vào cột `text`/`jsonb` giống như **cất đồ vào ngăn kéo của chính bàn mình**, bỏ bàn thì đồ đi theo. Lưu bằng LO giống như **gửi đồ ở kho ngoài và chỉ giữ tờ phiếu**. Xé phiếu (xóa dòng) thì kho vẫn còn đồ và vẫn chiếm đĩa.

Ví dụ minh họa:

```sql
CREATE TABLE demo_log (id serial PRIMARY KEY, json_request oid);

INSERT INTO demo_log (json_request)
VALUES (lo_from_bytea(0, convert_to('{"score":720}', 'UTF8')));

DELETE FROM demo_log;
VACUUM demo_log;

-- Object vẫn còn, không ai trỏ tới
SELECT count(*) FROM pg_largeobject_metadata;
```

Phải dọn thủ công bằng `lo_unlink(oid)` hoặc công cụ `vacuumlo`.

## 4. Vì sao dev không biết mình đang tạo LO

Với **Java + Hibernate/JPA**, đoạn code quen thuộc dưới đây trên PostgreSQL sẽ map sang cột `oid`:

```java
@Lob
private String jsonRequest;
```

Dev chỉ nghĩ "chuỗi dài thì thêm `@Lob`". Phía sau, framework gọi `lo_create`. Khi `delete()` hoặc job dọn log chạy, chỉ dòng bị xóa, large object ở lại. Với PHP/Python thì thường do gọi trực tiếp `lo_import`, `pg_lo_create` hoặc `connection.lobject()`.

## 5. Những tín hiệu nhận biết sớm

- Kích thước DB lớn hơn nhiều so với tổng các bảng nghiệp vụ.
- `pg_largeobject`, `pg_largeobject_metadata`, `pg_shdepend` nằm trong top bảng lớn nhất.
- `VACUUM` chạy xong nhưng dung lượng không giảm.
- Có cột kiểu `oid` ở bảng của app (lưu ý lọc `attnum > 0`, vì cột hệ thống `tableoid` cũng có kiểu `oid` ở mọi bảng).

Các truy vấn dùng để phát hiện:

```sql
-- Top đối tượng lớn nhất, gồm cả catalog
SELECT n.nspname, c.relname,
       pg_size_pretty(pg_total_relation_size(c.oid)) AS total
FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind = 'r'
ORDER BY pg_total_relation_size(c.oid) DESC LIMIT 15;

-- Cột nào đang lưu large object
SELECT c.relname, a.attname
FROM pg_attribute a
JOIN pg_class c ON c.oid = a.attrelid
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE a.atttypid = 'oid'::regtype AND a.attnum > 0
  AND c.relkind IN ('r','p')
  AND n.nspname NOT IN ('pg_catalog','information_schema','pg_toast')
  AND NOT a.attisdropped;

-- Số LO hiện có
SELECT count(*) FROM pg_largeobject_metadata;
```

## 6. Vì sao VACUUM không cứu được

| Lệnh | Tác dụng | Trả đĩa về OS? |
| --- | --- | --- |
| `VACUUM` thường | Dọn dead tuple để tái sử dụng nội bộ | Gần như không |
| `VACUUM FULL` | Ghi lại toàn bộ bảng sang file mới | Có, nhưng **cần chỗ trống ≈ dữ liệu còn sống** |
| `lo_unlink` / `vacuumlo` | Xóa large object mồ côi | Chưa, cần `VACUUM FULL` sau đó |

Vì ổ đã gần đầy, `VACUUM FULL` thất bại, và lần thất bại còn có thể để lại WAL hoặc file tạm. Đây là vòng luẩn quẩn: **cần chỗ trống để giải phóng chỗ trống**.

## 7. Cách xử lý đã áp dụng

**Bước 1: Lấy chỗ thở trước.** Dọn file ngoài DB (log cũ, backup cũ), kiểm tra `pg_wal` và replication slot bị bỏ quên.

**Bước 2: `TRUNCATE` 2 bảng log lớn** (`tbl_log_request_score`, `tbl_log_response_score`). `TRUNCATE` gần như không sinh WAL và trả đĩa ngay, khác với `DELETE` vừa tốn WAL vừa để lại bloat. Trước khi làm cần kiểm tra khóa ngoại và backup phần cần giữ.

**Bước 3: Dọn large object mồ côi.** Máy chủ không có sẵn `vacuumlo` (thuộc gói `postgresql-contrib`), nên có thể cài gói này hoặc dùng SQL thuần, vì bản chất `vacuumlo` chỉ là gọi `lo_unlink` cho các LO không được tham chiếu:

```sql
CREATE TEMP TABLE ref_lo AS
SELECT json_request AS oid FROM public.tbl_leadgen_customers_to_partner_log WHERE json_request IS NOT NULL
UNION
SELECT json_response FROM public.tbl_leadgen_customers_to_partner_log WHERE json_response IS NOT NULL;
CREATE INDEX ON ref_lo(oid);

-- Xóa theo lô nhỏ, lặp đến khi trả về 0
SELECT count(lo_unlink(s.oid)) FROM (
  SELECT m.oid FROM pg_largeobject_metadata m
  WHERE NOT EXISTS (SELECT 1 FROM ref_lo r WHERE r.oid = m.oid)
  LIMIT 1000
) s;
```

Lưu ý: nếu app vẫn đang ghi trong lúc dọn, LO mới chưa nằm trong `ref_lo` sẽ bị xóa nhầm, nên cần dừng app hoặc giới hạn theo OID đã chụp.

Nếu gần như toàn bộ LO là rác, phương án **dump bỏ large object (`pg_dump -B`) rồi restore sang DB mới** nhanh và sạch hơn nhiều so với `lo_unlink` hàng triệu object.

**Bước 4: Trả đĩa cho catalog** sau khi dọn xong:

```sql
VACUUM (FULL, VERBOSE) pg_largeobject;
VACUUM (FULL, VERBOSE) pg_largeobject_metadata;
VACUUM (FULL, VERBOSE) pg_shdepend;
```

## 8. Bài học và hành động phòng ngừa

### Dành cho Dev

1. **Không dùng Large Object để lưu JSON/log.** Dùng `jsonb` hoặc `text` (Postgres tự nén qua TOAST), hoặc object storage nếu là file lớn.
2. Với Hibernate, **cảnh giác với `@Lob`** trên PostgreSQL. Thay bằng `@Column(columnDefinition = "jsonb")` hoặc `text`.
3. Nếu bắt buộc dùng LO, **phải `lo_unlink` khi xóa dòng**.
4. Review schema do ORM tự sinh, đừng tin mặc định.

### Dành cho DBA/Vận hành

1. **Giám sát dung lượng theo từng bảng, gồm cả catalog**, không chỉ bảng nghiệp vụ.
2. **Cảnh báo ổ đĩa ở mức 70-80%**, để luôn đủ chỗ chạy `VACUUM FULL` hoặc `pg_repack`.
3. Cài sẵn `postgresql-contrib` để có `vacuumlo`, và cân nhắc extension `lo` với trigger `lo_manage` để tự dọn:

```sql
CREATE EXTENSION lo;
CREATE TRIGGER t_manage_lo
BEFORE UPDATE OR DELETE ON public.tbl_leadgen_customers_to_partner_log
FOR EACH ROW EXECUTE FUNCTION lo_manage(json_request, json_response);
```

4. **Bảng log phải có chính sách retention.** Partition theo ngày/tháng và `DROP PARTITION` thay vì `DELETE`.
5. **Rà soát index của bảng log.** Index 23 GB lớn hơn data 17 GB là dấu hiệu bloat hoặc index thừa.
6. Định kỳ chạy `vacuumlo` (hoặc script tương đương) nếu hệ thống còn dùng LO.

## 9. Checklist nhanh khi DB phình bất thường

- [ ] So sánh `pg_database_size` với tổng kích thước bảng nghiệp vụ.
- [ ] Liệt kê top đối tượng lớn nhất, **bao gồm `pg_catalog`**.
- [ ] Tìm cột kiểu `oid` trong schema của app (`attnum > 0`).
- [ ] Kiểm tra `pg_replication_slots` và thư mục `pg_wal`.
- [ ] Kiểm tra index không dùng (`pg_stat_user_indexes`, `idx_scan = 0`).
- [ ] Đảm bảo còn chỗ trống trước khi chạy bất kỳ lệnh `VACUUM FULL` nào.

## 10. Kết luận

Sự cố này không do dữ liệu nghiệp vụ nhiều, mà do một lựa chọn kỹ thuật nhỏ ở tầng ORM (`oid` thay vì `jsonb`/`text`) kết hợp với việc thiếu bước dọn dẹp khi xóa dữ liệu. Hai câu hỏi nên đặt ra mỗi khi review thiết kế bảng log: **"Dữ liệu này nằm ở đâu thật sự?"** và **"Khi xóa dòng, ai dọn phần còn lại?"**
