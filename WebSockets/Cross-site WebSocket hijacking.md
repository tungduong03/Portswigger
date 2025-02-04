# Cross-site WebSocket hijacking

## What is cross-site WebSocket hijacking?

Tấn công WebSocket cross-site (còn được gọi là tấn công WebSocket chéo nguồn) liên quan đến lỗ hổng làm giả yêu cầu chéo trang web (CSRF) trên bắt tay WebSocket. Lỗi này phát sinh khi yêu cầu bắt tay WebSocket chỉ dựa vào cookie HTTP để xử lý phiên và không chứa bất kỳ mã thông báo CSRF hoặc các giá trị không thể đoán trước nào.

Kẻ tấn công có thể tạo một trang web độc hại trên tên miền của chúng, thiết lập kết nối WebSocket xuyên trang web tới ứng dụng dễ bị tấn công. Ứng dụng sẽ xử lý kết nối trong bối cảnh phiên làm việc của người dùng nạn nhân với ứng dụng.





















