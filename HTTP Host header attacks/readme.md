# Mục đích của HTTP Host Header
HTTP Host Header được sử dụng để xác định thành phần back-end mà client muốn giao tiếp. 

Tại sao HTTP Host Header quan trọng?

Trước đây, mỗi địa chỉ IP thường chỉ lưu trữ nội dung cho một domain duy nhất. Tuy nhiên, do sự gia tăng của các giải pháp dựa trên cloud và sự thiếu hụt địa chỉ IPv4, hiện nay nhiều trang web và ứng dụng có thể chia sẻ chung một địa chỉ IP. 

Điều này thường xảy ra trong các trường hợp sau:

## Virtual Hosting (Lưu trữ ảo)

Một máy chủ web có thể lưu trữ nhiều website hoặc ứng dụng.

- Có thể là nhiều website thuộc cùng một chủ sở hữu.
- Hoặc các website khác nhau với các chủ sở hữu khác nhau trên cùng một nền tảng lưu trữ.

Hoạt động:

- Dù các website này có tên miền (domain) khác nhau, chúng vẫn chia sẻ chung một địa chỉ IP của máy chủ.

- Những website được lưu trữ theo cách này gọi là "virtual hosts" (host ảo).

## Định tuyến qua hệ thống trung gian (Routing traffic via an intermediary)

Các website được lưu trữ trên các máy chủ back-end riêng biệt.

Nhưng toàn bộ lưu lượng giữa client và các máy chủ được định tuyến qua một hệ thống trung gian, chẳng hạn:

- Load Balancer (Cân bằng tải)
- Reverse Proxy (Proxy ngược)
- Content Delivery Network (CDN)

Hoạt động:

- Các domain của website đều được ánh xạ (resolve) về một địa chỉ IP duy nhất của hệ thống trung gian.

- Hệ thống này cần sử dụng `Host Header` để biết yêu cầu nào cần gửi tới máy chủ back-end nào.

# What is an HTTP Host header attack?

HTTP Host header attack là việc kẻ tấn công lợi dụng việc back end xử lý đầu vào header `Host` không cẩn thận nên có thể lợi nó để tiêm các payload ngoài ý muốn

Cách hoạt động của HTTP Host Header Attack

- Nhiều ứng dụng web không biết rõ domain mà chúng đang được triển khai, trừ khi được cấu hình thủ công trong file cài đặt.

- Khi ứng dụng cần lấy domain hiện tại, ví dụ để tạo URL tuyệt đối trong email, nó có thể lấy thông tin từ giá trị `Host` Header. Ví dụ:

`<a href="https://_SERVER['HOST']/support">Contact support</a>`

Giá trị Host Header cũng có thể được sử dụng trong các tương tác giữa các hệ thống thuộc cơ sở hạ tầng của website.

Các lỗ hổng thường bị khai thác từ Host Header Attack

- Web Cache Poisoning (Đầu độc bộ nhớ đệm web):

- Lỗi logic nghiệp vụ (Business Logic Flaws):

- SSRF dựa trên định tuyến (Routing-based SSRF):

- Lỗ hổng phía máy chủ cổ điển (Classic Server-Side Vulnerabilities):

# Các phương pháp ngăn chặn tấn công HTTP Host Header:

## Bảo vệ URL tuyệt đối (Protect absolute URLs):

Khi phải sử dụng URL tuyệt đối:

- Yêu cầu domain hiện tại được chỉ định thủ công trong file cấu hình.

- Tham chiếu giá trị này thay vì sử dụng Host Header.

Cách tiếp cận này có thể loại bỏ các mối đe dọa như password reset poisoning.

## Xác thực Host Header (Validate the Host header):

Nếu bắt buộc phải sử dụng Host Header:

- Xác thực nó đúng cách bằng cách kiểm tra với một danh sách whitelist các domain được phép.

- Từ chối hoặc chuyển hướng bất kỳ yêu cầu nào chứa Host Header không hợp lệ.

Hướng dẫn thực hiện:
Kiểm tra tài liệu của framework bạn đang sử dụng. Ví dụ, trong framework Django, tùy chọn ALLOWED_HOSTS trong file settings có thể được sử dụng để định nghĩa danh sách domain hợp lệ.

Cách làm này giảm nguy cơ bị tấn công Host Header Injection.

## Không hỗ trợ ghi đè Host Header (Don't support Host override headers):

Đảm bảo rằng máy chủ không hỗ trợ các header bổ sung có thể bị khai thác, đặc biệt là `X-Forwarded-Host`.

Lưu ý rằng một số header này có thể được hỗ trợ mặc định bởi hệ thống.

## Whitelist các domain được phép (Whitelist permitted domains):

Để ngăn chặn tấn công định tuyến vào hạ tầng nội bộ, hãy cấu hình bộ cân bằng tải (load balancer) hoặc proxy ngược (reverse proxy) để chỉ chuyển tiếp các yêu cầu tới danh sách domain được phép.

## Thận trọng với virtual host nội bộ (Internal-only virtual hosts):

Khi sử dụng virtual hosting, tránh lưu trữ các website và ứng dụng nội bộ trên cùng một máy chủ với nội dung công khai.

Nếu không, kẻ tấn công có thể truy cập vào các domain nội bộ thông qua việc thao túng Host Header.







