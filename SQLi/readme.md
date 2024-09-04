Link: https://portswigger.net/web-security/sql-injection

# SQL INJECTION
Trong post này chúng ta sẽ tìm hiểu: 
- SQL injection là gì? (SQLi)
- Cách tìm và khai thác các loại lỗ hổng SQLi khác nhau
- Cách ngăn chặn SQLi

## SQL injection là gì? (SQLi)
SQL injection (SQLi) là một lỗ hổng bảo mật web cho phép kẻ tấn công can thiệp vào các truy vấn mà ứng dụng thực hiện đối với cơ sở dữ liệu của nó. Điều này có thể cho phép kẻ tấn công xem dữ liệu mà thông thường chúng không thể truy xuất được. Điều này có thể bao gồm dữ liệu thuộc về người dùng khác hoặc bất kỳ dữ liệu nào khác mà ứng dụng có thể truy cập. Trong nhiều trường hợp, kẻ tấn công có thể sửa đổi hoặc xóa dữ liệu này, gây ra những thay đổi liên tục đối với nội dung hoặc hành vi của ứng dụng.
Trong một số trường hợp, kẻ tấn công có thể nâng cấp cuộc tấn công SQL SQL để xâm phạm máy chủ cơ bản hoặc cơ sở hạ tầng phụ trợ khác. Nó cũng có thể cho phép họ thực hiện các cuộc tấn công từ chối dịch vụ.

## Tác động của một cuộc tấn công tiêm nhiễm SQL thành công là gì?

Một cuộc tấn công SQLi thành công có thể dẫn đến truy cập trái phép vào dữ liệu nhạy cảm, chẳng hạn như:
- Passwords
- Chi tiết credit card
- Thông tin cá nhân của người dùng

## Cách phát hiện lỗ hổng SQLi

Bạn có thể phát hiện SQLi theo cách thủ công bằng cách sử dụng một bộ thử nghiệm có hệ thống đối với mọi input trong ứng dụng. Để làm điều này, bạn thường sẽ submit:

- Ký tự trích dẫn đơn ' và tìm kiếm lỗi hoặc các điểm bất thường khác.

- Một số cú pháp dành riêng cho SQL đánh giá giá trị cơ sở (gốc) của điểm nhập và một số thông tin khác, đồng thời tìm kiếm sự khác biệt mang tính hệ thống trong các phản hồi của ứng dụng.

- Các điều kiện Boolean như OR 1=1 và OR 1=2, đồng thời tìm kiếm sự khác biệt trong phản hồi của ứng dụng.
- Tải trọng được thiết kế để kích hoạt độ trễ thời gian khi được thực thi trong truy vấn SQL và tìm kiếm sự khác biệt về thời gian cần thiết để phản hồi.
- Tải trọng OAST được thiết kế để kích hoạt tương tác mạng ngoài băng tần khi được thực thi trong truy vấn SQL và giám sát mọi tương tác phát sinh.

Ngoài ra, bạn có thể tìm thấy phần lớn các lỗ hổng SQL SQL một cách nhanh chóng và đáng tin cậy bằng cách sử dụng Burp Scanner.

## Inject SQL vào các phần khác nhau của truy vấn
Hầu hết các lỗ hổng SQL SQL xảy ra trong mệnh đề WHERE của truy vấn SELECT. Hầu hết những người thử nghiệm có kinh nghiệm đều quen thuộc với kiểu chèn SQL này.










