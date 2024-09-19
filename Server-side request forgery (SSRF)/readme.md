# Server-side request forgery (SSRF)
Trong phần này, chúng tôi giải thích giả mạo yêu cầu phía máy chủ (SSRF) là gì và mô tả một số ví dụ phổ biến. Chúng tôi cũng chỉ cho bạn cách tìm và khai thác các lỗ hổng SSRF.
## What is SSRF?
Giả mạo yêu cầu phía máy chủ là một lỗ hổng bảo mật web cho phép kẻ tấn công khiến ứng dụng phía máy chủ thực hiện các yêu cầu đến một vị trí ngoài ý muốn.\
Trong một cuộc tấn công SSRF điển hình, kẻ tấn công có thể khiến máy chủ tạo kết nối với các **dịch vụ chỉ dành cho nội bộ trong cơ sở hạ tầng của tổ chức.** Trong các trường hợp khác, họ có thể buộc máy chủ kết nối với các hệ thống bên ngoài tùy ý. Điều này có thể làm rò rỉ dữ liệu nhạy cảm, chẳng hạn như thông tin xác thực ủy quyền.\
![alt text](image.png)\
## What is the impact of SSRF attacks?
Một cuộc tấn công SSRF thành công thường có thể dẫn đến các hành động trái phép hoặc truy cập dữ liệu trong tổ chức. Điều này có thể nằm trong ứng dụng dễ bị tấn công hoặc trên các hệ thống phụ trợ khác mà ứng dụng có thể giao tiếp. Trong một số trường hợp, lỗ hổng SSRF có thể cho phép kẻ tấn công thực hiện lệnh tùy ý.\
Việc khai thác SSRF khiến kết nối với hệ thống bên thứ ba bên ngoài có thể dẫn đến các cuộc tấn công độc hại tiếp theo. Những lỗi này có thể xuất phát từ tổ chức lưu trữ ứng dụng dễ bị tấn công.
## Common SSRF attacks
Các cuộc tấn công SSRF thường khai thác các mối quan hệ tin cậy để leo thang cuộc tấn công từ ứng dụng dễ bị tấn công và thực hiện các hành động trái phép. Các mối quan hệ tin cậy này có thể tồn tại liên quan đến máy chủ hoặc liên quan đến các hệ thống phụ trợ khác trong cùng một tổ chức.
### SSRF attacks against the server
Trong một cuộc tấn công SSRF nhằm vào máy chủ, kẻ tấn công khiến ứng dụng thực hiện yêu cầu HTTP quay lại máy chủ đang lưu trữ ứng dụng, thông qua giao diện mạng loopback của nó. Điều này thường liên quan đến việc cung cấp URL có tên máy chủ như 127.0.0.1 (địa chỉ IP dành riêng trỏ đến bộ điều hợp loopback) hoặc localhost (tên thường được sử dụng cho cùng một bộ điều hợp).\
Ví dụ: hãy tưởng tượng một ứng dụng mua sắm cho phép người dùng xem liệu một mặt hàng có còn hàng ở một cửa hàng cụ thể hay không. Để cung cấp thông tin chứng khoán, ứng dụng phải truy vấn các API REST phụ trợ khác nhau. Nó thực hiện điều này bằng cách chuyển URL đến điểm cuối API back-end có liên quan thông qua yêu cầu HTTP front-end. Khi người dùng xem trạng thái tồn kho của một mặt hàng, trình duyệt của họ sẽ đưa ra yêu cầu sau:
```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```
Điều này khiến máy chủ đưa ra yêu cầu tới URL được chỉ định, truy xuất trạng thái tồn kho và trả lại trạng thái này cho người dùng.\
Trong ví dụ này, kẻ tấn công có thể sửa đổi yêu cầu chỉ định URL cục bộ cho máy chủ:
```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://localhost/admin
```
Máy chủ tìm nạp nội dung của URL `/admin` và trả lại cho người dùng.\
Kẻ tấn công có thể truy cập URL /admin, nhưng chức năng quản trị thường chỉ những người dùng được xác thực mới có thể truy cập được. Điều này có nghĩa là kẻ tấn công sẽ không thấy bất cứ điều gì đáng quan tâm. Tuy nhiên, nếu yêu cầu tới URL /admin xuất phát từ máy cục bộ thì các biện pháp kiểm soát truy cập thông thường sẽ bị bỏ qua. Ứng dụng cấp quyền truy cập đầy đủ vào chức năng quản trị vì yêu cầu dường như bắt nguồn từ một vị trí đáng tin cậy.

----------------------
Ví dụ: https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-localhost

Bài này ở request check stock ta thấy parameter có giá trị là 1 URL, vì vậy ta biết đây là chỗ để inject, từ đó ta thay bằng `http://localhost/admin` thì sẽ có kết quả trả về là page của admin.

----------------------

Tại sao các ứng dụng lại hoạt động theo cách này và ngầm tin tưởng các yêu cầu đến từ máy cục bộ? Điều này có thể phát sinh vì nhiều lý do:
- Việc kiểm tra kiểm soát truy cập có thể được triển khai trong một thành phần khác nằm phía trước máy chủ ứng dụng. Khi kết nối được thực hiện trở lại máy chủ, việc kiểm tra sẽ bị bỏ qua.
- Với mục đích khắc phục thảm họa, ứng dụng có thể cho phép truy cập quản trị mà không cần đăng nhập đối với bất kỳ người dùng nào đến từ máy cục bộ. Điều này cung cấp một cách để quản trị viên khôi phục hệ thống nếu họ mất thông tin đăng nhập. Điều này giả định rằng chỉ người dùng hoàn toàn đáng tin cậy mới đến trực tiếp từ máy chủ.
- Giao diện quản trị có thể nghe trên một số cổng khác với ứng dụng chính và người dùng có thể không truy cập trực tiếp được.

Những mối quan hệ tin cậy kiểu này, trong đó các yêu cầu xuất phát từ máy cục bộ được xử lý khác với các yêu cầu thông thường, thường khiến SSRF trở thành một lỗ hổng nghiêm trọng.

### SSRF attacks against other back-end systems
Trong một số trường hợp, máy chủ ứng dụng có thể tương tác với các hệ thống phụ trợ mà người dùng không thể truy cập trực tiếp. Các hệ thống này thường có địa chỉ IP riêng không thể định tuyến. Các hệ thống back-end thường được bảo vệ bởi cấu trúc liên kết mạng nên chúng thường có tình trạng bảo mật yếu hơn. Trong nhiều trường hợp, hệ thống back-end nội bộ chứa chức năng nhạy cảm có thể được truy cập mà không cần xác thực bởi bất kỳ ai có thể tương tác với hệ thống.\
Trong ví dụ trước, hãy tưởng tượng có giao diện quản trị tại URL https://192.168.0.68/admin. Kẻ tấn công có thể gửi yêu cầu sau để khai thác lỗ hổng SSRF và truy cập vào giao diện quản trị:
```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://192.168.0.68/admin
```

---
Ví dụ: https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-backend-system

Bài này ta được biết localhost là dải IP: 192.168.0.x:8080/admin ta sẽ cần brute-force để tìm x này, từ đó SSRF thành công

---
## Circumventing common SSRF defenses
Người ta thường thấy các ứng dụng có hành vi SSRF cùng với các biện pháp bảo vệ nhằm ngăn chặn việc khai thác độc hại. Thông thường, những biện pháp phòng vệ này có thể bị phá vỡ.
### SSRF with blacklist-based input filters
Một số ứng dụng chặn đầu vào có chứa tên máy chủ như `127.0.0.1` và `localhost` hoặc các URL nhạy cảm như `/admin`. Trong tình huống này, bạn thường có thể phá vỡ bộ lọc bằng các kỹ thuật sau:
- Sử dụng địa chỉ IP thay thế là `127.0.0.1`, chẳng hạn như `2130706433`, `017700000001` hoặc `127.1`.
- Đăng ký tên miền của riêng bạn có độ phân giải thành `127.0.0.1`. Bạn có thể sử dụng `spofed.burpcollaborator.net` cho mục đích này.
- Làm xáo trộn các chuỗi bị chặn bằng cách sử dụng mã hóa URL hoặc biến thể chữ hoa chữ thường.
- Cung cấp URL mà bạn kiểm soát, URL này sẽ chuyển hướng đến URL mục tiêu. Hãy thử sử dụng các mã chuyển hướng khác nhau cũng như các giao thức khác nhau cho URL mục tiêu. Ví dụ: việc chuyển từ URL `http:` sang `https:` trong quá trình chuyển hướng đã được chứng minh là bỏ qua một số bộ lọc chống SSRF.

Ví dụ: https://portswigger.net/web-security/ssrf/lab-ssrf-with-blacklist-filter

Bài này filter `127.0.0.1` và `localhost` và `admin` ta có thể thay thế bằng `127.1` và `aDmin` để thoát filter:
![alt text](image-1.png)

### SSRF with whitelist-based input filters
Một số ứng dụng chỉ cho phép đầu vào trùng với whitelist. Bộ lọc có thể tìm kiếm kết quả khớp ở đầu đầu vào hoặc có trong đó. Bạn có thể bỏ qua bộ lọc này bằng cách khai thác sự không nhất quán trong phân tích cú pháp URL.\
Đặc tả URL chứa một số tính năng có thể bị bỏ qua khi URL triển khai phân tích cú pháp và xác thực đặc biệt bằng phương pháp này:
- Bạn có thể nhúng thông tin xác thực vào URL trước tên máy chủ, sử dụng ký tự `@`. Ví dụ:\
`https://expected-host:fakepassword@evil-host`
- Bạn có thể sử dụng ký tự `#` để biểu thị một đoạn URL. Ví dụ:\
`https://evil-host#expected-host`
- Bạn có thể tận dụng hệ thống phân cấp đặt tên DNS để đặt thông tin đầu vào bắt buộc vào tên DNS đủ điều kiện mà bạn kiểm soát. Ví dụ:\
`https://expected-host.evil-host`
- Bạn có thể mã hóa các ký tự URL để gây nhầm lẫn cho mã phân tích cú pháp URL. Điều này đặc biệt hữu ích nếu mã triển khai bộ lọc xử lý các ký tự được mã hóa URL khác với mã thực hiện yêu cầu HTTP phía sau. Bạn cũng có thể thử mã hóa kép các ký tự; một số máy chủ giải mã URL đệ quy đầu vào mà chúng nhận được, điều này có thể dẫn đến sự khác biệt hơn nữa.
- Bạn có thể sử dụng kết hợp các kỹ thuật này với nhau.

Ví dụ: https://portswigger.net/web-security/ssrf/lab-ssrf-with-whitelist-filter

Whitelist bắt buộc nó bắt đầu bằng `stock.weliketoshop.net`:\
![alt text](image-2.png)

Dùng page: `https://portswigger.net/web-security/ssrf/url-validation-bypass-cheat-sheet` để tạo payload và dùng intruder để thử: \
![alt text](image-4.png)\
![alt text](image-3.png)

![alt text](image-5.png)

### Bypassing SSRF filters via open redirection
Đôi khi có thể vượt qua các biện pháp phòng vệ dựa trên bộ lọc bằng cách khai thác lỗ hổng chuyển hướng mở.\
Trong ví dụ trước, hãy tưởng tượng URL do người dùng gửi được xác thực nghiêm ngặt để ngăn chặn việc khai thác hành vi SSRF có mục đích xấu. Tuy nhiên, ứng dụng có URL được phép chứa lỗ hổng chuyển hướng mở. Với điều kiện API được sử dụng để thực hiện yêu cầu HTTP phía sau hỗ trợ chuyển hướng, bạn có thể tạo một URL thỏa mãn bộ lọc và dẫn đến yêu cầu được chuyển hướng đến mục tiêu phía sau mong muốn.\
Ví dụ: ứng dụng chứa lỗ hổng chuyển hướng mở trong đó URL sau:\
`/product/nextProduct?currentProductId=6&path=http://evil-user.net`\
trả về một chuyển hướng đến:\
`http://evil-user.net`\
Bạn có thể tận dụng lỗ hổng chuyển hướng mở để vượt qua bộ lọc URL và khai thác lỗ hổng SSRF như sau:
```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin
```
Việc khai thác SSRF này hoạt động vì trước tiên ứng dụng xác thực rằng URL stockAPI được cung cấp nằm trên một miền được phép. Sau đó, ứng dụng sẽ yêu cầu URL được cung cấp để kích hoạt chuyển hướng mở. Nó tuân theo sự chuyển hướng và đưa ra yêu cầu tới URL nội bộ mà kẻ tấn công lựa chọn.

Ví dụ: https://portswigger.net/web-security/ssrf/lab-ssrf-filter-bypass-via-open-redirection

Ở bài này check stock không còn dùng URL nữa, mà nó chỉ cho phép là endpoint của API. Nên ta cần tìm endpoint nào có parameter có thể thêm URL, thì ở tính năng nextProduct có thể chèn URL, ta đưa endpoint này vào check stock để truy xuất:\
![alt text](image-7.png)\
![alt text](image-6.png)

Tại vì sao ta không dùng luôn nextProduct để tấn công? Lý do là vì nextProduct chỉ dùng path phía sau parameter `path` để đi đến trang tiếp theo, nếu dùng URL ta bắt buộc đó là URL có thể phân giải toàn cục, vì nó ngắt với trang cũ, nên khi dùng IP cục bộ thì nó không thể tìm thấy trang cần tìm, mặt khác ở check stock ta vẫn có Host để xác định trang để tải. 

## Blind SSRF vulnerabilities
Lỗ hổng blind SSRF xảy ra nếu bạn có thể khiến ứng dụng đưa ra yêu cầu HTTP back-end tới một URL được cung cấp nhưng phản hồi từ yêu cầu back-end không được trả về trong phản hồi front-end của ứng dụng.\
SSRF blind khó khai thác hơn nhưng đôi khi dẫn đến việc thực thi mã từ xa hoàn toàn trên máy chủ hoặc các back-end component khác.

Đọc thêm [Tìm và khai thác lỗ hổng blind SSRF](<Blind SSRF vulnerabilities.md>)

## Finding hidden attack surface for SSRF vulnerabilities
Nhiều lỗ hổng giả mạo yêu cầu phía máy chủ rất dễ tìm thấy vì lưu lượng thông thường của ứng dụng liên quan đến các tham số yêu cầu chứa URL đầy đủ. Các ví dụ khác về SSRF khó xác định hơn.
### Partial URLs in requests
Đôi khi, ứng dụng chỉ đặt tên máy chủ hoặc **một phần đường dẫn URL** vào tham số yêu cầu. Sau đó, giá trị được gửi sẽ được kết hợp phía máy chủ vào một URL đầy đủ được yêu cầu. Nếu giá trị được nhận dạng dễ dàng dưới dạng tên máy chủ hoặc đường dẫn URL thì bề mặt tấn công tiềm ẩn có thể rõ ràng. Tuy nhiên, khả năng khai thác dưới dạng SSRF đầy đủ có thể bị hạn chế do bạn không kiểm soát toàn bộ URL được yêu cầu.
### URLs within data formats
Một số ứng dụng truyền dữ liệu ở các định dạng có thông số kỹ thuật cho phép bao gồm các URL có thể được trình phân tích cú pháp dữ liệu yêu cầu đối với định dạng đó. Một ví dụ rõ ràng về điều này là định dạng dữ liệu XML, định dạng này đã được sử dụng rộng rãi trong các ứng dụng web để truyền dữ liệu có cấu trúc từ máy khách đến máy chủ. Khi một ứng dụng chấp nhận dữ liệu ở định dạng XML và phân tích cú pháp dữ liệu đó, nó có thể dễ bị tiêm XXE. Nó cũng có thể dễ bị SSRF thông qua XXE. Chúng tôi sẽ đề cập đến vấn đề này chi tiết hơn khi xem xét các lỗ hổng chèn XXE.
### SSRF via the Referer header
Một số ứng dụng sử dụng phần mềm phân tích phía máy chủ để theo dõi khách truy cập. Phần mềm này thường ghi lại tiêu đề `Referer` trong các yêu cầu để có thể theo dõi các liên kết đến. Thông thường, phần mềm phân tích sẽ truy cập bất kỳ URL nào của bên thứ ba xuất hiện trong tiêu đề `Referer`. Điều này thường được thực hiện để phân tích nội dung của các trang web giới thiệu, bao gồm cả văn bản liên kết được sử dụng trong các liên kết đến. Do đó, tiêu đề `Referer` thường là bề mặt tấn công hữu ích cho các lỗ hổng SSRF.

