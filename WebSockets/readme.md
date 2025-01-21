# Testing for WebSockets security vulnerabilities

Trong phần này, chúng tôi sẽ giải thích cách thao tác các kết nối và tin nhắn WebSocket, mô tả các loại lỗ hổng bảo mật có thể phát sinh với WebSocket và đưa ra một số ví dụ về cách khai thác lỗ hổng WebSocket.

## WebSockets

WebSockets được sử dụng rộng rãi trong các ứng dụng web hiện đại. Chúng được khởi tạo qua HTTP và cung cấp các kết nối lâu dài với giao tiếp không đồng bộ theo cả hai hướng. 

WebSockets được sử dụng cho mọi mục đích, bao gồm thực hiện hành động của người dùng và truyền thông tin nhạy cảm. Hầu như bất kỳ lỗ hổng bảo mật web nào phát sinh với HTTP thông thường cũng có thể phát sinh liên quan đến giao tiếp WebSockets.

## Manipulating WebSocket traffic

Việc tìm lỗ hổng bảo mật WebSockets thường liên quan đến việc thao túng chúng theo những cách mà ứng dụng không ngờ tới. Bạn có thể thực hiện việc này bằng Burp Suite.

### Intercepting and modifying WebSocket messages

Bạn có thể sử dụng Burp Proxy để chặn và sửa đổi tin nhắn WebSocket như sau:

- Open Burp's browser.
- Duyệt đến chức năng ứng dụng sử dụng WebSockets. Bạn có thể xác định WebSockets đang được sử dụng bằng cách sử dụng ứng dụng và tìm kiếm các mục xuất hiện trong tab lịch sử WebSockets trong Burp Proxy.
- Trong tab Intercept của Burp Proxy, hãy đảm bảo rằng tính năng chặn đã được bật.
- Khi tin nhắn WebSocket được gửi từ trình duyệt hoặc máy chủ, tin nhắn đó sẽ được hiển thị trong tab `Intercept` để bạn xem hoặc sửa đổi. Nhấn nút `Forward` để chuyển tiếp tin nhắn.

note: Bạn có thể cấu hình xem tin nhắn từ máy khách đến máy chủ hay từ máy chủ đến máy khách có bị chặn trong Burp Proxy hay không. Thực hiện việc này trong hộp thoại Cài đặt, trong cài đặt quy tắc chặn WebSocket.

### Replaying and generating new WebSocket messages

Ngoài việc chặn và sửa đổi tin nhắn WebSocket ngay lập tức, bạn có thể phát lại từng tin nhắn và tạo tin nhắn mới. Bạn có thể thực hiện việc này bằng Burp Repeater:

- Trong Burp Proxy, hãy chọn một tin nhắn trong lịch sử WebSockets hoặc trong tab Intercept và chọn "Send to Repeater" từ menu.
- Trong Burp Repeater, giờ đây bạn có thể chỉnh sửa tin nhắn đã chọn và gửi đi nhiều lần.
- Bạn có thể nhập tin nhắn mới và gửi theo bất kỳ hướng nào, tới máy khách hoặc máy chủ.
- Trong bảng "History" trong Burp Repeater, bạn có thể xem lịch sử các tin nhắn đã được truyền qua kết nối WebSocket. Điều này bao gồm các tin nhắn bạn đã tạo trong Burp Repeater cũng như bất kỳ tin nhắn nào được tạo bởi trình duyệt hoặc máy chủ thông qua cùng một kết nối.
- Nếu bạn muốn chỉnh sửa và gửi lại bất kỳ tin nhắn nào trong bảng lịch sử, bạn có thể thực hiện bằng cách chọn tin nhắn và chọn "Edit and resend" từ menu.

### Manipulating WebSocket connections

Ngoài việc thao tác các thông điệp WebSocket, đôi khi cần phải thao tác bắt tay WebSocket để thiết lập kết nối.

Có nhiều tình huống khác nhau mà việc thao tác bắt tay WebSocket có thể cần thiết:
- Nó có thể giúp bạn tiếp cận được nhiều bề mặt tấn công hơn.
- Một số cuộc tấn công có thể khiến kết nối của bạn bị mất nên bạn cần thiết lập kết nối mới.
- Token hoặc dữ liệu khác trong yêu cầu bắt tay ban đầu có thể đã cũ và cần được cập nhật.

Bạn có thể thao tác bắt tay WebSocket bằng Burp Repeater:
- Gửi tin nhắn WebSocket đến Burp Repeater
- Trong Burp Repeater, hãy nhấp vào biểu tượng bút chì bên cạnh URL WebSocket. Thao tác này sẽ mở ra một trình hướng dẫn cho phép bạn kết nối với một WebSocket đã kết nối hiện có, sao chép một WebSocket đã kết nối hoặc kết nối lại với một WebSocket đã ngắt kết nối.





















