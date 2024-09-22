# XML entities
Trong phần này, chúng tôi sẽ giải thích một số **tính năng chính của XML** có liên quan đến việc hiểu các lỗ hổng XXE.
## What is XML?
XML là viết tắt của "extensible markup language". XML là ngôn ngữ được thiết kế để lưu trữ và truyền tải dữ liệu. Giống như HTML, XML sử dụng cấu trúc thẻ và dữ liệu dạng cây. Không giống như HTML, XML không sử dụng các thẻ được xác định trước và do đó các thẻ có thể được đặt tên để mô tả dữ liệu. Trước đó trong lịch sử web, XML đã thịnh hành dưới dạng định dạng truyền dữ liệu ("X" trong "AJAX" là viết tắt của "XML"). Nhưng mức độ phổ biến của nó hiện đã giảm do định dạng JSON.
## What are XML entities?
Các thực thể XML là một cách thể hiện một mục dữ liệu trong một tài liệu XML, thay vì sử dụng chính dữ liệu đó. Nhiều thực thể khác nhau được tích hợp sẵn theo đặc tả của ngôn ngữ XML. Ví dụ: các thực thể `&lt;` và `&gt;` đại diện cho các ký tự `<` và `>`. Đây là các siêu ký tự được sử dụng để biểu thị các thẻ XML và do đó thường phải được biểu diễn bằng các thực thể của chúng khi chúng xuất hiện trong dữ liệu.
## What is document type definition?
Định nghĩa loại tài liệu XML (document type definition - DTD) chứa các khai báo có thể xác định cấu trúc của tài liệu XML, các loại giá trị dữ liệu mà nó có thể chứa và các mục khác. DTD được khai báo trong phần tử `DOCTYPE` tùy chọn ở đầu tài liệu XML. DTD có thể hoàn toàn độc lập bên trong tài liệu (được gọi là "internal DTD") hoặc có thể được tải từ nơi khác (được gọi là "external DTD") hoặc có thể kết hợp cả hai.
## What are XML custom entities?
XML cho phép các thực thể tùy chỉnh được xác định trong DTD. Ví dụ:\
`<!DOCTYPE foo [ <!ENTITY myentity "my entity value" > ]>`\
Định nghĩa này có nghĩa là mọi cách sử dụng tham chiếu thực thể `&myentity;` trong tài liệu XML sẽ được thay thế bằng giá trị được xác định: "my entity value".
## What are XML external entities?
Các thực thể bên ngoài XML là một loại thực thể tùy chỉnh có định nghĩa nằm bên ngoài DTD nơi chúng được khai báo.\
Việc khai báo một thực thể bên ngoài sử dụng từ khóa `SYSTEM` và phải chỉ định một URL từ đó giá trị của thực thể sẽ được tải. Ví dụ:\
`<!DOCTYPE foo [ <!ENTITY ext SYSTEM "http://normal-website.com" > ]>`\
URL có thể sử dụng giao thức `file://` và do đó các thực thể bên ngoài có thể được tải từ tệp. Ví dụ:\
`<!DOCTYPE foo [ <!ENTITY ext SYSTEM "file:///path/to/file" > ]>`\
Các thực thể bên ngoài XML cung cấp các phương tiện chính để phát sinh các cuộc tấn công thực thể XML bên ngoài.




















