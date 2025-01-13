# Client-side prototype pollution vulnerabilities

Trong phần này, bạn sẽ học cách tìm lỗ hổng prototype pollution phía client trong môi trường thực tế. Để củng cố sự hiểu biết về cách các lỗ hổng này hoạt động, chúng tôi sẽ hướng dẫn cách thực hiện thủ công cũng như cách sử dụng công cụ DOM Invader để tự động hóa phần lớn quá trình này. 

## Tìm các nguồn prototype pollution phía client thủ công
Việc tìm nguồn prototype pollution thủ công chủ yếu dựa vào phương pháp thử và sai. Nói ngắn gọn, bạn cần thử các cách khác nhau để thêm một thuộc tính tuỳ ý vào `Object.prototype` cho đến khi tìm được nguồn hoạt động.

Khi kiểm tra các lỗ hổng phía client, quá trình này bao gồm các bước chính sau:







