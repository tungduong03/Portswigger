# Web LLM attacks

Các cuộc tấn công LLM có thể bao gồm:

- Truy xuất dữ liệu mà LLM có quyền truy cập.
- Kích hoạt các hành vi gây hại thông qua API.
- Kích hoạt các cuộc tấn công vào người dùng và hệ thống khác truy vấn LLM.

# Phát hiện 

- Xác định các đầu vào của LLM, bao gồm cả đầu vào trực tiếp (như prompt) và gián tiếp (như dữ liệu train).

- Tìm hiểu xem LLM có quyền truy cập vào dữ liệu và API nào.

- Thăm dò bề mặt tấn công mới để tìm lỗ hổng.

# Khai thác LLM APIs, functions, and plugins

LLM thường được lưu trữ bởi các nhà cung cấp bên thứ ba chuyên dụng. Một trang web có thể cung cấp cho LLM của bên thứ ba quyền truy cập vào chức năng cụ thể của nó bằng cách mô tả API cục bộ để LLM sử dụng.

Ví dụ, LLM hỗ trợ khách hàng có thể có quyền truy cập vào các API quản lý người dùng, đơn hàng và kho.

## How LLM APIs work

Khi gọi API bên ngoài, một số LLM có thể yêu cầu máy khách gọi endpoint hàm riêng biệt (thực chất là API riêng) để tạo các yêu cầu hợp lệ có thể gửi đến các API đó. Quy trình cho việc này có thể giống như sau:

- Client gọi LLM theo prompt của người dùng.
- LLM phát hiện rằng một hàm cần được gọi và trả về một đối tượng JSON chứa các đối số tuân theo lược đồ API bên ngoài.
- Client gọi hàm với các đối số được cung cấp.
- Client xử lý phản hồi của chức năng.
- Client gọi lại LLM, thêm phản hồi hàm dưới dạng một thông báo mới.
- LLM gọi API bên ngoài bằng hàm response.
- LLM tóm tắt kết quả của lệnh gọi API này gửi lại cho người dùng.

Quy trình công việc này có thể có những tác động về bảo mật, vì LLM thực sự đang gọi các API bên ngoài thay mặt cho người dùng nhưng người dùng có thể không biết rằng các API này đang được gọi. Trong trường hợp lý tưởng, người dùng sẽ được cung cấp bước xác nhận trước khi LLM gọi API bên ngoài.







