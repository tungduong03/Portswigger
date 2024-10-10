# Cross-origin resource sharing (CORS)
## What is CORS (cross-origin resource sharing)?
Chia sẻ tài nguyên nhiều nguồn gốc (CORS) là một cơ chế trình duyệt cho phép truy cập có kiểm soát vào các tài nguyên nằm bên ngoài một miền nhất định. Nó mở rộng và bổ sung tính linh hoạt cho chính sách cùng nguồn gốc (SOP). Tuy nhiên, nó cũng tiềm ẩn nguy cơ xảy ra các cuộc tấn công giữa các miền nếu chính sách CORS của trang web được định cấu hình và triển khai kém. CORS không phải là biện pháp bảo vệ chống lại các cuộc tấn công có nguồn gốc chéo, chẳng hạn như giả mạo yêu cầu chéo trang (CSRF).
## Same-origin policy
Chính sách cùng nguồn gốc là một đặc tả hạn chế về nhiều nguồn gốc nhằm giới hạn khả năng trang web tương tác với các tài nguyên bên ngoài miền nguồn. Chính sách cùng nguồn gốc đã được xác định từ nhiều năm trước để ứng phó với các tương tác giữa các miền độc hại tiềm ẩn, chẳng hạn như một trang web đánh cắp dữ liệu riêng tư từ một trang web khác. Nó thường cho phép một miền đưa ra yêu cầu cho các miền khác nhưng không truy cập được các phản hồi.
## Relaxation of the same-origin policy
Chính sách cùng nguồn gốc rất hạn chế và do đó nhiều cách tiếp cận khác nhau đã được đưa ra để tránh những hạn chế đó. Nhiều trang web tương tác với tên miền phụ hoặc trang web của bên thứ ba theo cách yêu cầu quyền truy cập đầy đủ từ nhiều nguồn gốc. Có thể nới lỏng có kiểm soát chính sách cùng nguồn gốc bằng cách sử dụng chia sẻ tài nguyên nhiều nguồn gốc (CORS).\
Giao thức chia sẻ tài nguyên nhiều nguồn gốc sử dụng một bộ tiêu đề HTTP xác định nguồn gốc web đáng tin cậy và các thuộc tính liên quan, chẳng hạn như liệu quyền truy cập được xác thực có được phép hay không. Chúng được kết hợp trong một trao đổi tiêu đề giữa trình duyệt và trang web có nguồn gốc chéo mà nó đang cố truy cập.
## Vulnerabilities arising from CORS configuration issues
Nhiều trang web hiện đại sử dụng CORS để cho phép truy cập từ các tên miền phụ và bên thứ ba đáng tin cậy. Việc triển khai CORS của họ có thể chứa lỗi hoặc quá dễ dãi để đảm bảo mọi thứ hoạt động và điều này có thể dẫn đến các lỗ hổng có thể khai thác.
### Server-generated ACAO header from client-specified Origin header
Một số ứng dụng cần cung cấp quyền truy cập vào một số miền khác. Việc duy trì danh sách các miền được phép đòi hỏi nỗ lực liên tục và bất kỳ sai sót nào cũng có nguy cơ phá vỡ chức năng. Vì vậy, một số ứng dụng sử dụng con đường dễ dàng để cho phép truy cập từ bất kỳ miền nào khác một cách hiệu quả.\
Một cách để thực hiện điều này là đọc tiêu đề Origin từ các yêu cầu và bao gồm tiêu đề phản hồi nêu rõ rằng nguồn gốc yêu cầu được phép. Ví dụ, hãy xem xét một ứng dụng nhận được yêu cầu sau:





