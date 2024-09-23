# XML external entity (XXE) injection
Trong phần này, chúng tôi sẽ giải thích XML external entity injection là gì, mô tả một số ví dụ phổ biến, giải thích cách tìm và khai thác các kiểu chèn XXE khác nhau cũng như tóm tắt cách ngăn chặn các cuộc tấn công chèn XXE.
## What is XML external entity injection?
XML external entity injection (còn được gọi là XXE) là một lỗ hổng bảo mật web cho phép kẻ tấn công can thiệp vào quá trình xử lý dữ liệu XML của ứng dụng. Nó thường cho phép kẻ tấn công xem các tệp trên hệ thống tệp của máy chủ ứng dụng và tương tác với bất kỳ hệ thống phụ trợ hoặc bên ngoài nào mà chính ứng dụng có thể truy cập.\
Trong một số trường hợp, kẻ tấn công có thể leo thang cuộc tấn công XXE để xâm phạm máy chủ cơ bản hoặc cơ sở hạ tầng phụ trợ khác, bằng cách lợi dụng lỗ hổng XXE để thực hiện các cuộc tấn công giả mạo yêu cầu phía máy chủ (SSRF).\
![alt text](image.png)

## How do XXE vulnerabilities arise?
Một số ứng dụng sử dụng định dạng XML để truyền dữ liệu giữa trình duyệt và máy chủ. Các ứng dụng thực hiện việc này hầu như luôn sử dụng thư viện chuẩn hoặc API nền tảng để xử lý dữ liệu XML trên máy chủ. Lỗ hổng XXE phát sinh do đặc tả XML chứa nhiều tính năng nguy hiểm tiềm tàng khác nhau và các trình phân tích cú pháp tiêu chuẩn hỗ trợ các tính năng này ngay cả khi chúng thường không được ứng dụng sử dụng.\
Đọc thêm [Tìm hiểu về định dạng XML, DTD và các thực thể bên ngoài](<XML entities.md>)\
Các thực thể bên ngoài XML là một loại thực thể XML tùy chỉnh có các giá trị được xác định được tải từ bên ngoài DTD mà chúng được khai báo. Các thực thể bên ngoài đặc biệt thú vị từ góc độ bảo mật vì chúng cho phép xác định một thực thể dựa trên nội dung của đường dẫn tệp hoặc URL.
## What are the types of XXE attacks?
Có nhiều loại tấn công XXE khác nhau:\
- **Exploiting XXE to retrieve files**, trong đó một thực thể bên ngoài được xác định có chứa nội dung của tệp và được trả về trong phản hồi của ứng dụng.
- **Exploiting XXE to perform SSRF attacks**, trong đó một thực thể bên ngoài được xác định dựa trên URL tới hệ thống phụ trợ.
- **Exploiting blind XXE exfiltrate data out-of-band**, nơi dữ liệu nhạy cảm được truyền từ máy chủ ứng dụng đến hệ thống mà kẻ tấn công kiểm soát.
- **Exploiting blind XXE to retrieve data via error messages**, nơi kẻ tấn công có thể kích hoạt thông báo lỗi phân tích cú pháp chứa dữ liệu nhạy cảm.

## Exploiting XXE to retrieve files
Để thực hiện cuộc tấn công chèn XXE nhằm truy xuất một tệp tùy ý từ hệ thống tệp của máy chủ, bạn cần sửa đổi XML đã gửi theo hai cách:
- Giới thiệu (hoặc chỉnh sửa) phần tử DOCTYPE xác định thực thể bên ngoài chứa đường dẫn đến tệp.
- Chỉnh sửa giá trị dữ liệu trong XML được trả về trong phản hồi của ứng dụng để sử dụng thực thể bên ngoài đã xác định.

Ví dụ: giả sử một ứng dụng mua sắm kiểm tra mức tồn kho của sản phẩm bằng cách gửi XML sau tới máy chủ:\
```
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck><productId>381</productId></stockCheck>
```
Ứng dụng không thực hiện biện pháp phòng vệ cụ thể nào trước các cuộc tấn công XXE, vì vậy bạn có thể khai thác lỗ hổng XXE để truy xuất tệp `/etc/passwd` bằng cách gửi tải trọng XXE sau:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```
Tải trọng XXE này xác định một thực thể bên ngoài `&xxe;` có giá trị là nội dung của tệp `/etc/passwd` và sử dụng thực thể trong giá trị `ProductId`. Điều này khiến phản hồi của ứng dụng bao gồm nội dung của tệp:
```
Invalid product ID: root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
```
Note: Với các lỗ hổng XXE trong thế giới thực, thường sẽ có một số lượng lớn giá trị dữ liệu trong XML được gửi, bất kỳ giá trị nào trong số đó có thể được sử dụng trong phản hồi của ứng dụng. Để kiểm tra một cách có hệ thống các lỗ hổng XXE, thông thường bạn sẽ cần kiểm tra từng nút dữ liệu trong XML riêng lẻ bằng cách sử dụng thực thể đã xác định của mình và xem liệu nó có xuất hiện trong phản hồi hay không.

Ví dụ: https://portswigger.net/web-security/xxe/lab-exploiting-xxe-to-retrieve-files

![alt text](image-1.png)

## Exploiting XXE to perform SSRF attacks
Ngoài việc truy xuất dữ liệu nhạy cảm, tác động chính khác của các cuộc tấn công XXE là chúng có thể được sử dụng để thực hiện giả mạo yêu cầu phía máy chủ (SSRF). Đây là một lỗ hổng nghiêm trọng tiềm ẩn trong đó ứng dụng phía máy chủ có thể bị lợi dụng để thực hiện các yêu cầu HTTP tới bất kỳ URL nào mà máy chủ có thể truy cập.\
Để khai thác lỗ hổng XXE nhằm thực hiện cuộc tấn công SSRF, bạn cần xác định một thực thể XML bên ngoài bằng URL mà bạn muốn nhắm mục tiêu và sử dụng thực thể đã xác định trong một giá trị dữ liệu. Nếu bạn có thể sử dụng thực thể đã xác định trong giá trị dữ liệu được trả về trong phản hồi của ứng dụng thì bạn sẽ có thể xem phản hồi từ URL trong phản hồi của ứng dụng và do đó có được sự tương tác hai chiều với hệ thống phụ trợ. Nếu không, bạn sẽ chỉ có thể thực hiện các cuộc tấn công SSRF mù quáng (vẫn có thể gây ra hậu quả nghiêm trọng).\
Trong ví dụ XXE sau, thực thể bên ngoài sẽ khiến máy chủ tạo yêu cầu HTTP back-end tới hệ thống nội bộ trong cơ sở hạ tầng của tổ chức:\
`<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.vulnerable-website.com/"> ]>`

Ví dụ: https://portswigger.net/web-security/xxe/lab-exploiting-xxe-to-perform-ssrf

Tương tự bài trên thay nội dung cần đọc:\
![alt text](image-2.png)

Phần `Invalid product ID:` là cố định, `lastest` là path, ta thêm path đó, rồi lần lượt request đến cuối cùng: \
![alt text](image-3.png)

## Blind XXE vulnerabilities
Nhiều trường hợp lỗ hổng XXE bị mù. Điều này có nghĩa là ứng dụng không trả về giá trị của bất kỳ thực thể bên ngoài nào được xác định trong phản hồi của nó và do đó không thể truy xuất trực tiếp các tệp phía máy chủ.\
Các lỗ hổng XXE mù vẫn có thể bị phát hiện và khai thác nhưng cần có các kỹ thuật nâng cao hơn. Đôi khi, bạn có thể sử dụng các kỹ thuật ngoài băng tần để tìm lỗ hổng và khai thác chúng để lấy dữ liệu. Và đôi khi bạn có thể gây ra lỗi phân tích cú pháp XML dẫn đến tiết lộ dữ liệu nhạy cảm trong các thông báo lỗi.

Đọc thêm [Tìm và khai thác lỗ hổng XXE mù](<blind XXE vulnerabilities.md>)

## Finding hidden attack surface for XXE injection - Tìm nơi để tấn công XXE
Bề mặt tấn công đối với các lỗ hổng chèn XXE là hiển nhiên trong nhiều trường hợp, vì lưu lượng HTTP thông thường của ứng dụng bao gồm các yêu cầu chứa dữ liệu ở định dạng XML. Trong các trường hợp khác, bề mặt tấn công ít được nhìn thấy hơn. Tuy nhiên, đôi khi, bạn sẽ thấy bề mặt tấn công XXE trong các yêu cầu không chứa bất kỳ XML nào.
### XInclude attacks
Một số ứng dụng nhận dữ liệu do khách hàng gửi, nhúng dữ liệu đó ở phía máy chủ vào tài liệu XML rồi phân tích cú pháp tài liệu đó. Một ví dụ về điều này xảy ra khi dữ liệu do khách hàng gửi được đặt vào một yêu cầu `SOAP`, sau đó được xử lý bởi `SOAP` service.\
Trong tình huống này, bạn không thể thực hiện cuộc tấn công XXE cổ điển vì bạn không kiểm soát toàn bộ tài liệu XML và do đó không thể xác định hoặc sửa đổi phần tử `DOCTYPE`. Tuy nhiên, bạn có thể sử dụng `XInclude` thay thế. `XInclude` là một phần của đặc tả XML cho phép xây dựng một tài liệu XML từ các tài liệu phụ. Bạn có thể thực hiện một cuộc tấn công `XInclude` bên trong bất kỳ giá trị dữ liệu nào trong tài liệu XML, do đó cuộc tấn công có thể được thực hiện trong các tình huống mà bạn chỉ kiểm soát một mục dữ liệu duy nhất được đặt vào tài liệu XML phía máy chủ.\
Để thực hiện một cuộc tấn công `XInclude`, bạn cần tham chiếu không gian tên `XInclude` và cung cấp đường dẫn đến tệp mà bạn muốn đưa vào. Ví dụ:
```
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>
```
Ví dụ: https://portswigger.net/web-security/xxe/lab-xinclude-attack

Ta thấy ở đây thông tin sẽ được trích xuất từ 2 biến `productID` và `storeId` vậy nên ta ko thể dùng như cách trước. 

![alt text](image-14.png)

### XXE attacks via file upload
Một số ứng dụng cho phép người dùng tải lên các tệp sau đó được xử lý phía máy chủ. Một số định dạng tệp phổ biến sử dụng XML hoặc chứa các thành phần phụ XML. Ví dụ về các định dạng dựa trên `XML` là các định dạng tài liệu văn phòng như `DOCX` và các định dạng hình ảnh như `SVG`.\
Ví dụ: một ứng dụng có thể cho phép người dùng tải hình ảnh lên và xử lý hoặc xác thực những hình ảnh này trên máy chủ sau khi chúng được tải lên. Ngay cả khi ứng dụng mong muốn nhận được định dạng như `PNG` hoặc `JPEG`, thư viện xử lý hình ảnh đang được sử dụng có thể hỗ trợ hình ảnh `SVG`. Vì định dạng `SVG` sử dụng XML nên kẻ tấn công có thể gửi hình ảnh `SVG` độc hại và do đó tiếp cận bề mặt tấn công ẩn để tìm lỗ hổng XXE.

Ví dụ: https://portswigger.net/web-security/xxe/lab-xxe-via-file-upload

Với bài này ta có tính năng upload avatar, và ta nghĩ đến việc SVG cũng là 1 loại image:\
![alt text](image-15.png)

Ta sẽ tạo 1 file SVG với nội dung:
```
<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]><svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>
```
![alt text](image-16.png)

![alt text](image-17.png)

### XXE attacks via modified content type
Hầu hết các yêu cầu POST sử dụng loại nội dung mặc định được tạo bởi các biểu mẫu HTML, chẳng hạn như `application/x-www-form-urlencoded`. Một số trang web mong muốn nhận được yêu cầu ở định dạng này nhưng sẽ chấp nhận các loại nội dung khác, bao gồm cả XML.\
Ví dụ: nếu một yêu cầu bình thường có nội dung sau:
```
POST /action HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 7

foo=bar
```
Sau đó, bạn có thể gửi yêu cầu sau với kết quả tương tự:
```
POST /action HTTP/1.0
Content-Type: text/xml
Content-Length: 52

<?xml version="1.0" encoding="UTF-8"?><foo>bar</foo>
```
Nếu ứng dụng chấp nhận các yêu cầu chứa XML trong nội dung thư và phân tích nội dung nội dung dưới dạng XML thì bạn có thể tiếp cận bề mặt tấn công XXE ẩn chỉ bằng cách định dạng lại các yêu cầu để sử dụng định dạng XML.
## How to find and test for XXE vulnerabilities
Kiểm tra thủ công các lỗ hổng XXE thường bao gồm:
- Kiểm tra khả năng truy xuất tệp bằng cách xác định một thực thể bên ngoài dựa trên tệp hệ điều hành phổ biến và sử dụng thực thể đó trong dữ liệu được trả về trong phản hồi của ứng dụng.
- Kiểm tra các lỗ hổng XXE mù bằng cách xác định một thực thể bên ngoài dựa trên URL tới hệ thống mà bạn kiểm soát và giám sát các tương tác với hệ thống đó. Burp Collaborator là sự lựa chọn hoàn hảo cho mục đích này.
- Kiểm tra khả năng dễ bị tổn thương của dữ liệu không phải XML do người dùng cung cấp trong tài liệu XML phía máy chủ bằng cách sử dụng cuộc tấn công `XInclude` để cố truy xuất tệp hệ điều hành phổ biến.

Hãy nhớ rằng XML chỉ là một định dạng truyền dữ liệu. Đảm bảo rằng bạn cũng kiểm tra mọi chức năng dựa trên XML để tìm các lỗ hổng khác như XSS và SQL SQL. Bạn có thể cần mã hóa tải trọng của mình bằng cách sử dụng các chuỗi thoát XML để tránh vi phạm cú pháp, nhưng bạn cũng có thể sử dụng điều này để làm xáo trộn cuộc tấn công của mình nhằm vượt qua khả năng phòng thủ yếu.

## How to prevent XXE vulnerabilities
Hầu như tất cả các lỗ hổng XXE đều phát sinh do thư viện phân tích cú pháp XML của ứng dụng hỗ trợ các tính năng XML nguy hiểm tiềm tàng mà ứng dụng không cần hoặc không có ý định sử dụng. Cách dễ dàng và hiệu quả nhất để ngăn chặn các cuộc tấn công XXE là vô hiệu hóa các tính năng đó.\
Nói chung, việc vô hiệu hóa tính năng của các thực thể bên ngoài và vô hiệu hóa hỗ trợ cho `XInclude` là đủ. Điều này thường có thể được thực hiện thông qua các tùy chọn cấu hình hoặc bằng cách ghi đè hành vi mặc định theo chương trình. Tham khảo tài liệu dành cho thư viện hoặc API phân tích cú pháp XML của bạn để biết chi tiết về cách tắt các chức năng không cần thiết.








