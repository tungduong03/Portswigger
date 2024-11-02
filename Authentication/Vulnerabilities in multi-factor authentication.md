# Vulnerabilities in multi-factor authentication

Multi-factor authentication (MFA) là phương pháp bảo mật sử dụng nhiều hơn một yếu tố để xác thực người dùng, thường kết hợp giữa **biết** (như mật khẩu) và **có** (như mã xác minh). Điều này giúp tăng cường bảo mật, tuy nhiên, nếu MFA được triển khai không đúng cách, nó có thể bị bỏ qua hoặc khai thác bởi các lỗ hổng bảo mật.

## Two-factor authentication tokens

Mã xác minh thường được gửi đến thiết bị vật lý của người dùng như mã thông báo RSA, ứng dụng di động (ví dụ **Google Authenticator**) hoặc qua **SMS**. Các mã được tạo trực tiếp từ thiết bị bảo mật chuyên dụng có độ an toàn cao hơn. Tuy nhiên, mã qua **SMS** dễ bị tấn công như **hoán đổi SIM**, trong đó kẻ tấn công đánh cắp thẻ SIM của nạn nhân để nhận mã xác minh. Ngoài ra, mã gửi qua SMS có thể bị chặn bởi các cuộc tấn công mạng.

## Bypassing two-factor authentication

MFA có thể bị bỏ qua nếu trang web cho phép người dùng truy cập vào các trang "đã đăng nhập" trước khi hoàn tất bước xác thực mã. Trong trường hợp này, kẻ tấn công chỉ cần truy cập trực tiếp vào các trang yêu cầu xác thực sau khi người dùng đã nhập mật khẩu nhưng chưa hoàn thành việc nhập mã xác minh, từ đó bỏ qua bước MFA hoàn toàn. 

---

Ví dụ: https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass

Sau khi đăng nhập nó sẽ bắt mình nhập 4 kí tự gửi về mail: 
![alt text](image-17.png)

Nhưng thực sự ở đây đã ở trạng thái login rồi nên chỉ cần 1 link chuyển về `/my-account` là có thể thoát khỏi phần điền 4 kí tự này:
![alt text](image-18.png)

## Flawed two-factor verification logic
Đôi khi logic thiếu sót trong xác thực hai yếu tố có nghĩa là sau khi người dùng hoàn thành bước đăng nhập ban đầu, trang web không xác minh đầy đủ rằng chính người dùng đó đang hoàn thành bước thứ hai.\
Ví dụ người dùng đăng nhập bằng thông tin đăng nhập thông thường của họ ở bước đầu tiên như sau:
```
POST /login-steps/first HTTP/1.1
Host: vulnerable-website.com
...
username=carlos&password=qwerty
```
Sau đó, họ được chỉ định một cookie liên quan đến tài khoản của họ trước khi được chuyển sang bước thứ hai của quy trình đăng nhập:
```
HTTP/1.1 200 OK
Set-Cookie: account=carlos

GET /login-steps/second HTTP/1.1
Cookie: account=carlos
```
Khi gửi mã xác minh, yêu cầu sẽ sử dụng cookie này để xác định tài khoản nào người dùng đang cố truy cập:
```
POST /login-steps/second HTTP/1.1
Host: vulnerable-website.com
Cookie: account=carlos
...
verification-code=123456
```
Trong trường hợp này, kẻ tấn công có thể đăng nhập bằng thông tin đăng nhập của riêng họ nhưng sau đó thay đổi giá trị của cookie tài khoản thành bất kỳ tên người dùng tùy ý nào khi gửi mã xác minh.\
```
POST /login-steps/second HTTP/1.1
Host: vulnerable-website.com
Cookie: account=victim-user
...
verification-code=123456
```
Điều này cực kỳ nguy hiểm nếu kẻ tấn công sau đó có thể ép buộc mã xác minh vì nó sẽ cho phép họ đăng nhập vào tài khoản của người dùng tùy ý hoàn toàn dựa trên tên người dùng của họ. Họ thậm chí sẽ không bao giờ cần biết mật khẩu của người dùng.

---

Ví dụ: https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic

Ở bài này ta sẽ sửa cookie cần xác thực thành `Carlos` rồi brute-force mã code mfa đến khi đúng:\
Lưu ý cần gửi request: GET /login2 với cookie `Carlos` để nó sinh ra `mfa` trước khi brute-force:
![alt text](image-19.png)\
![alt text](image-20.png)

## Brute-forcing 2FA verification codes

Các mã xác minh 2FA thường chỉ có 4 hoặc 6 chữ số, điều này khiến cho việc brute-force mã trở nên khả thi nếu không có cơ chế bảo vệ đủ mạnh. Một số trang web cố gắng ngăn chặn brute-force bằng cách đăng xuất người dùng sau một số lần nhập sai mã. Tuy nhiên, kẻ tấn công có thể tự động hóa quy trình này bằng cách sử dụng công cụ như **Burp Intruder** hoặc **Turbo Intruder** để brute-force hàng ngàn mã trong một khoảng thời gian ngắn. 

Nếu trang web không có các biện pháp bảo vệ chống lại brute-force hiệu quả, điều này sẽ dễ dàng dẫn đến việc kẻ tấn công có thể đột nhập tài khoản thông qua việc brute-force mã xác minh 2FA.

---

Ví dụ: https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-bypass-using-a-brute-force-attack

Cần thực hiện vài lần để ra kết quả, vì thời gian brute-force khá lâu nên có thể server sẽ đổi `mfa` đi:
![alt text](image-36.png)





