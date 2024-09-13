Link: https://portswigger.net/web-security/authentication

# Authentication vulnerabilities
Về mặt khái niệm, lỗ hổng xác thực rất dễ hiểu. Tuy nhiên, chúng thường rất quan trọng vì mối quan hệ rõ ràng giữa xác thực và bảo mật.\
Lỗ hổng xác thực có thể cho phép kẻ tấn công truy cập vào dữ liệu và chức năng nhạy cảm. Họ cũng phơi bày bề mặt tấn công bổ sung để khai thác thêm. Vì lý do này, điều quan trọng là phải tìm hiểu cách xác định và khai thác các lỗ hổng xác thực cũng như cách vượt qua các biện pháp bảo vệ thông thường.\
Trong phần này, chúng tôi giải thích:\
- Các cơ chế xác thực phổ biến nhất được sử dụng bởi các trang web.
- Các lỗ hổng tiềm ẩn trong các cơ chế này.
- Lỗ hổng cố hữu trong các cơ chế xác thực khác nhau.
- Các lỗ hổng điển hình xuất hiện do việc triển khai không đúng cách.
- Cách bạn có thể tạo cơ chế xác thực của riêng mình mạnh mẽ nhất có thể.

![alt text](image.png)

## What is authentication?
Xác thực là quá trình xác minh danh tính của người dùng hoặc khách hàng. Các trang web có khả năng bị lộ cho bất kỳ ai kết nối với Internet. Điều này làm cho cơ chế xác thực mạnh mẽ trở thành một phần không thể thiếu để bảo mật web hiệu quả.\
Có ba loại xác thực chính:
- Thông tin nào đó bạn `biết` (`know`), chẳng hạn như mật khẩu hoặc câu trả lời cho câu hỏi bảo mật. Đôi khi chúng được gọi là "yếu tố kiến ​​thức".
- Thứ bạn `có` (`have`), Đây là một vật thể như điện thoại di động hoặc mã thông báo bảo mật. Đôi khi chúng được gọi là "yếu tố sở hữu".
- Một cái gì đó bạn đang (`are`) hoặc làm (`do`). Ví dụ: sinh trắc học hoặc mô hình hành vi của bạn. Đôi khi chúng được gọi là "yếu tố vốn có".

Cơ chế xác thực dựa trên một loạt công nghệ để xác minh một hoặc nhiều yếu tố này.
## What is the difference between authentication and authorization?
**Xác thực** là quá trình **xác minh** rằng người dùng chính là người mà họ tuyên bố. **Ủy quyền** liên quan đến việc xác minh xem người dùng **có được phép làm điều gì đó hay không**.\
Ví dụ: xác thực xác định xem ai đó đang cố truy cập trang web có tên người dùng `Carlos123` có thực sự là người đã tạo tài khoản hay không.\
Khi `Carlos123` được xác thực, quyền của họ sẽ xác định những gì họ được phép làm. Ví dụ: họ có thể được ủy quyền truy cập thông tin cá nhân của người dùng khác hoặc thực hiện các hành động như xóa tài khoản của người dùng khác.
## Lỗ hổng xác thực phát sinh như thế nào?
Hầu hết các lỗ hổng trong cơ chế xác thực xảy ra theo một trong hai cách:
- Các cơ chế xác thực yếu vì chúng không thể bảo vệ đầy đủ trước các cuộc tấn công vũ phu.
- Lỗi logic hoặc mã hóa kém trong quá trình triển khai cho phép kẻ tấn công bỏ qua hoàn toàn các cơ chế xác thực. Điều này đôi khi được gọi là "broken authentication".

Trong nhiều lĩnh vực phát triển web, các lỗi logic khiến trang web hoạt động không mong muốn, điều này có thể có hoặc không phải là vấn đề bảo mật. Tuy nhiên, vì xác thực rất quan trọng đối với bảo mật nên rất có thể logic xác thực bị thiếu sót sẽ khiến trang web gặp phải các vấn đề bảo mật.
## Tác động của xác thực dễ bị tổn thương là gì?
Tác động của lỗ hổng xác thực có thể nghiêm trọng. Nếu kẻ tấn công bỏ qua xác thực hoặc đột nhập vào tài khoản của người dùng khác, chúng có quyền truy cập vào tất cả dữ liệu và chức năng mà tài khoản bị xâm phạm có. Nếu họ có thể xâm phạm tài khoản có đặc quyền cao, chẳng hạn như quản trị viên hệ thống, họ có thể có toàn quyền kiểm soát toàn bộ ứng dụng và có khả năng giành được quyền truy cập vào cơ sở hạ tầng nội bộ.\
Ngay cả việc xâm phạm tài khoản có đặc quyền thấp vẫn có thể cấp cho kẻ tấn công quyền truy cập vào dữ liệu mà lẽ ra chúng không nên có, chẳng hạn như thông tin doanh nghiệp nhạy cảm về mặt thương mại. Ngay cả khi tài khoản không có quyền truy cập vào bất kỳ dữ liệu nhạy cảm nào, nó vẫn có thể cho phép kẻ tấn công truy cập các trang bổ sung, cung cấp thêm bề mặt tấn công. Thông thường, các cuộc tấn công có mức độ nghiêm trọng cao không thể thực hiện được từ các trang có thể truy cập công khai nhưng chúng có thể xảy ra từ một trang nội bộ.
## Lỗ hổng trong cơ chế xác thực
Hệ thống xác thực của trang web thường bao gồm một số cơ chế riêng biệt có thể xảy ra lỗ hổng. Một số lỗ hổng có thể áp dụng được trên tất cả các bối cảnh này. Những người khác cụ thể hơn về chức năng được cung cấp.\
Chúng ta sẽ xem xét kỹ hơn một số lỗ hổng phổ biến nhất trong các lĩnh vực sau:
- [Lỗ hổng trong đăng nhập dựa trên mật khẩu](<Vulnerabilities in password-based login.md>)
- [Lỗ hổng trong xác thực đa yếu tố](<Vulnerabilities in multi-factor authentication.md>)
- [Lỗ hổng trong các cơ chế xác thực khác](<Vulnerabilities in other authentication mechanisms.md>)

Một số phòng thí nghiệm yêu cầu bạn liệt kê tên người dùng và brute-force mật khẩu. Để giúp bạn thực hiện quy trình này, chúng tôi cung cấp danh sách rút gọn gồm [username](username.txt) và [password](password.txt) ứng viên mà bạn nên sử dụng để giải quyết các bài thí nghiệm.

## Lỗ hổng trong cơ chế xác thực của bên thứ ba
Nếu bạn thích hack các cơ chế xác thực và bạn đã hoàn thành xác thực chính của chúng tôi, bạn có thể muốn thử phòng thí nghiệm xác thực OAuth của chúng tôi.\
https://portswigger.net/web-security/oauth\
Có khoảng 6 labs

## Cách bảo mật cơ chế xác thực của bạn
Trong phần này, chúng tôi sẽ nói về cách bạn có thể ngăn chặn một số lỗ hổng mà chúng tôi đã thảo luận xảy ra trong cơ chế xác thực của bạn.\
Xác thực là một chủ đề phức tạp và, như chúng tôi đã chứng minh, thật không may là rất dễ để các điểm yếu và sai sót lọt vào. Việc phác thảo mọi biện pháp khả thi mà bạn có thể thực hiện để bảo vệ trang web của riêng mình rõ ràng là không thể thực hiện được. Tuy nhiên, có một số nguyên tắc chung mà bạn phải luôn tuân theo.
### Take care with user credentials
Ngay cả các cơ chế xác thực mạnh mẽ nhất cũng không hiệu quả nếu bạn vô tình tiết lộ bộ thông tin xác thực đăng nhập hợp lệ cho kẻ tấn công. Không cần phải nói rằng bạn không bao giờ nên gửi bất kỳ dữ liệu đăng nhập nào qua các kết nối không được mã hóa. Mặc dù bạn có thể đã triển khai HTTPS cho các yêu cầu đăng nhập của mình nhưng hãy đảm bảo rằng bạn thực thi điều này bằng cách chuyển hướng mọi yêu cầu HTTP đã thử sang HTTPS.\
Bạn cũng nên kiểm tra trang web của mình để đảm bảo rằng không có tên người dùng hoặc địa chỉ email nào được tiết lộ thông qua hồ sơ có thể truy cập công khai hoặc được phản ánh trong phản hồi HTTP chẳng hạn.
### Don't count on users for security
Các biện pháp xác thực nghiêm ngặt thường yêu cầu người dùng của bạn phải nỗ lực thêm. Bản chất con người khiến tất cả nhưng không thể tránh khỏi việc một số người dùng sẽ tìm cách tự cứu lấy nỗ lực này. Vì vậy, bạn cần thực thi hành vi an toàn bất cứ khi nào có thể.\
Ví dụ rõ ràng nhất là thực hiện chính sách mật khẩu hiệu quả. Một số chính sách truyền thống hơn bị thất bại vì mọi người đưa mật khẩu có thể dự đoán được của riêng họ vào chính sách. Thay vào đó, có thể hiệu quả hơn nếu triển khai một số loại trình kiểm tra mật khẩu đơn giản, cho phép người dùng thử nghiệm mật khẩu và cung cấp phản hồi về độ mạnh của mật khẩu trong thời gian thực. Một ví dụ phổ biến là thư viện JavaScript `zxcvbn`, được phát triển bởi Dropbox. Bằng cách chỉ cho phép những mật khẩu được trình kiểm tra mật khẩu đánh giá cao, bạn có thể thực thi việc sử dụng mật khẩu an toàn hiệu quả hơn so với các chính sách truyền thống.
### Prevent username enumeration
Kẻ tấn công sẽ dễ dàng phá vỡ cơ chế xác thực của bạn hơn đáng kể nếu bạn tiết lộ rằng người dùng tồn tại trên hệ thống. Thậm chí có một số trường hợp nhất định, do tính chất của trang web, bản thân việc biết rằng một người cụ thể có tài khoản đã là thông tin nhạy cảm.\
Bất kể tên người dùng đã thử có hợp lệ hay không, điều quan trọng là phải sử dụng các thông báo lỗi chung, giống hệt nhau và đảm bảo rằng chúng thực sự giống nhau. Bạn phải luôn trả về cùng một mã trạng thái HTTP cho mỗi yêu cầu đăng nhập và cuối cùng, làm cho thời gian phản hồi trong các tình huống khác nhau càng không thể phân biệt được càng tốt.
### Implement robust brute-force protection
Do việc xây dựng một cuộc tấn công vũ phu có thể đơn giản như thế nào, điều quan trọng là phải đảm bảo rằng bạn thực hiện các bước để ngăn chặn hoặc ít nhất là làm gián đoạn mọi nỗ lực đăng nhập bằng vũ lực.\
Một trong những phương pháp hiệu quả hơn là thực hiện giới hạn tỷ lệ người dùng dựa trên IP một cách nghiêm ngặt. Điều này sẽ bao gồm các biện pháp ngăn chặn kẻ tấn công thao túng địa chỉ IP rõ ràng của họ. Tốt nhất, bạn nên yêu cầu người dùng hoàn thành bài kiểm tra CAPTCHA với mỗi lần đăng nhập sau khi đạt đến một giới hạn nhất định.\
Hãy nhớ rằng điều này không đảm bảo sẽ loại bỏ hoàn toàn mối đe dọa cưỡng bức vũ phu. Tuy nhiên, việc làm cho quá trình trở nên tẻ nhạt và thủ công nhất có thể sẽ làm tăng khả năng bất kỳ kẻ tấn công nào có ý định từ bỏ và tìm kiếm mục tiêu nhẹ nhàng hơn.
### Triple-check your verification logic
Như các phòng thí nghiệm của chúng tôi đã chứng minh, các lỗi logic đơn giản rất dễ xâm nhập vào mã mà trong trường hợp xác thực có khả năng làm tổn hại hoàn toàn trang web và người dùng của bạn. Việc kiểm tra kỹ lưỡng mọi logic xác minh hoặc xác thực để loại bỏ sai sót là chìa khóa tuyệt đối để xác thực mạnh mẽ. Cuối cùng, một cuộc kiểm tra có thể được bỏ qua cũng chẳng tốt hơn là không có cuộc kiểm tra nào cả.
### Don't forget supplementary functionality
Đảm bảo không chỉ tập trung vào các trang đăng nhập trung tâm mà bỏ qua chức năng bổ sung liên quan đến xác thực. Điều này đặc biệt quan trọng trong trường hợp kẻ tấn công có thể tự do đăng ký tài khoản của mình và khám phá chức năng này. Hãy nhớ rằng việc đặt lại hoặc thay đổi mật khẩu cũng là một bề mặt tấn công hợp lệ như cơ chế đăng nhập chính và do đó, phải mạnh mẽ như nhau.
### Implement proper multi-factor authentication
Mặc dù xác thực đa yếu tố có thể không thực tế đối với mọi trang web, nhưng khi được thực hiện đúng cách, nó sẽ an toàn hơn nhiều so với việc chỉ đăng nhập bằng mật khẩu. Hãy nhớ rằng việc xác minh nhiều trường hợp của cùng một yếu tố không phải là xác thực đa yếu tố thực sự. Gửi mã xác minh qua email về cơ bản chỉ là một hình thức xác thực một yếu tố dài dòng hơn.\
2FA dựa trên SMS đang xác minh về mặt kỹ thuật hai yếu tố (thứ bạn biết và thứ bạn có). Tuy nhiên, ví dụ, khả năng lạm dụng thông qua việc hoán đổi SIM có nghĩa là hệ thống này có thể không đáng tin cậy.\
Lý tưởng nhất là nên triển khai 2FA bằng thiết bị hoặc ứng dụng chuyên dụng tạo mã xác minh trực tiếp. Vì chúng được xây dựng nhằm mục đích cung cấp bảo mật nên chúng thường an toàn hơn.\
Cuối cùng, giống như logic xác thực chính, hãy đảm bảo rằng logic trong quá trình kiểm tra 2FA của bạn hợp lý để không thể dễ dàng bỏ qua.




