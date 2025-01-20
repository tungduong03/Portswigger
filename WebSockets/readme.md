# Testing for WebSockets security vulnerabilities

Trong phần này, chúng tôi sẽ giải thích cách thao tác các kết nối và tin nhắn WebSocket, mô tả các loại lỗ hổng bảo mật có thể phát sinh với WebSocket và đưa ra một số ví dụ về cách khai thác lỗ hổng WebSocket.

## WebSockets

WebSockets được sử dụng rộng rãi trong các ứng dụng web hiện đại. Chúng được khởi tạo qua HTTP và cung cấp các kết nối lâu dài với giao tiếp không đồng bộ theo cả hai hướng. 

WebSockets được sử dụng cho mọi mục đích, bao gồm thực hiện hành động của người dùng và truyền thông tin nhạy cảm. Hầu như bất kỳ lỗ hổng bảo mật web nào phát sinh với HTTP thông thường cũng có thể phát sinh liên quan đến giao tiếp WebSockets.

## Manipulating WebSocket traffic

Việc tìm lỗ hổng bảo mật WebSockets thường liên quan đến việc thao túng chúng theo những cách mà ứng dụng không ngờ tới. Bạn có thể thực hiện việc này bằng Burp Suite.

### Intercepting and modifying WebSocket messages
























