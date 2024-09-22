# NoSQL databases
Cơ sở dữ liệu NoSQL lưu trữ và truy xuất dữ liệu ở định dạng khác với các bảng quan hệ SQL truyền thống. Chúng được thiết kế để xử lý khối lượng lớn dữ liệu phi cấu trúc hoặc bán cấu trúc. Vì vậy, chúng thường có ít ràng buộc quan hệ và kiểm tra tính nhất quán hơn so với SQL, đồng thời khẳng định những lợi ích đáng kể về khả năng mở rộng, tính linh hoạt và hiệu suất.\
Giống như cơ sở dữ liệu SQL, người dùng tương tác với dữ liệu trong cơ sở dữ liệu NoSQL bằng cách sử dụng các truy vấn được ứng dụng chuyển đến cơ sở dữ liệu. Tuy nhiên, các cơ sở dữ liệu NoSQL khác nhau sử dụng nhiều ngôn ngữ truy vấn thay vì một tiêu chuẩn chung như SQL (Ngôn ngữ truy vấn có cấu trúc). Đây có thể là ngôn ngữ truy vấn tùy chỉnh hoặc ngôn ngữ phổ biến như XML hoặc JSON.

# NoSQL database models
Có rất nhiều cơ sở dữ liệu NoSQL. Để phát hiện các lỗ hổng trong cơ sở dữ liệu NoSQL, việc hiểu được framework và ngôn ngữ mô hình sẽ giúp ích.\
Một số loại cơ sở dữ liệu NoSQL phổ biến bao gồm:
- Lưu trữ database - Những dữ liệu này lưu trữ trong các tài liệu bán cấu trúc, linh hoạt. Chúng thường sử dụng các định dạng như `JSON`, `BSON` và `XML` và được truy vấn bằng `API` hoặc ngôn ngữ truy vấn. Ví dụ bao gồm `MongoDB` và `Couchbase`.
- Lưu trữ **key-value** - Những dữ liệu này lưu trữ ở định dạng **key-value**. Mỗi trường dữ liệu được liên kết với một chuỗi khóa duy nhất. Các giá trị được truy xuất dựa trên khóa duy nhất. Ví dụ bao gồm `Redis` và `Amazon DynamoDB`.
- Lưu trữ **Wide-column** - Chúng sắp xếp dữ liệu liên quan thành các họ cột linh hoạt thay vì các hàng truyền thống. Ví dụ bao gồm `Apache Cassandra` và `Apache HBase`.
- **Graph databases** - Chúng sử dụng các nút để lưu trữ các thực thể dữ liệu và các cạnh để lưu trữ mối quan hệ giữa các thực thể. Ví dụ bao gồm `Neo4j` và `Amazon Neptune`.












