# Server-side prototype pollution

Protype pollution có thể dẫn đến RCE

## Tại sao prototype pollution ở server side khó phát hiện hơn?

Vì một số lý do, ô nhiễm nguyên mẫu phía máy chủ thường khó phát hiện hơn so với biến thể phía máy khách:

- No source code access: Code ở server sẽ không thể nhìn thấy nên đây là black box, khó để tìm sink
- Lack of developer tools: 
- The DoS problem: Việc gây ô nhiễm thành công các đối tượng trong môi trường phía máy chủ bằng các thuộc tính thực thường làm hỏng chức năng của ứng dụng hoặc khiến máy chủ ngừng hoạt động hoàn toàn. Ngay cả khi bạn xác định được lỗ hổng, việc khai thác lỗ hổng này cũng rất khó khăn khi về cơ bản bạn đã phá hỏng cả trang web trong quá trình này.
- Pollution persistence: Khi bạn làm ô nhiễm một nguyên mẫu phía máy chủ, thay đổi này sẽ tồn tại trong toàn bộ vòng đời của quy trình Node và bạn không có cách nào để thiết lập lại nó.

Trong các phần sau, chúng tôi sẽ đề cập đến một số kỹ thuật không phá hủy cho phép bạn kiểm tra ô nhiễm nguyên mẫu phía máy chủ một cách an toàn bất chấp những hạn chế này.

## Detecting server-side prototype pollution via polluted property reflection

Một cái bẫy dễ mắc phải đối với các nhà phát triển là quên hoặc bỏ qua thực tế rằng vòng lặp `for...in` của JavaScript sẽ lặp lại tất cả các thuộc tính có thể liệt kê của một đối tượng, bao gồm cả những thuộc tính mà đối tượng đó thừa hưởng thông qua chuỗi nguyên mẫu.

```js
const myObject = { a: 1, b: 2 };

// pollute the prototype with an arbitrary property
Object.prototype.foo = 'bar';

// confirm myObject doesn't have its own foo property
myObject.hasOwnProperty('foo'); // false

// list names of properties of myObject
for(const propertyKey in myObject){
    console.log(propertyKey);
}

// Output: a, b, foo
```

Điều này cũng áp dụng cho mảng, trong đó vòng lặp `for...in` trước tiên sẽ lặp qua từng chỉ mục, về cơ bản chỉ là một khóa thuộc tính số ẩn bên dưới, trước khi chuyển sang bất kỳ thuộc tính nào được kế thừa.

```js
const myArray = ['a','b'];

Object.prototype.foo = 'bar';

for(const arrayKey in myArray){
    console.log(arrayKey);
}

// Output: 0, 1, foo
```

Trong cả hai trường hợp, nếu sau đó ứng dụng bao gồm các thuộc tính được trả về trong phản hồi, thì đây có thể là một cách đơn giản để thăm dò ô nhiễm nguyên mẫu phía máy chủ.

Các yêu cầu `POST` hoặc `PUT` gửi dữ liệu JSON đến ứng dụng hoặc API là những ứng cử viên chính cho loại hành vi này vì máy chủ thường phản hồi bằng biểu diễn JSON của đối tượng mới hoặc đối tượng đã cập nhật. Trong trường hợp này, bạn có thể thử làm ô nhiễm `Object.prototype` toàn cục bằng một thuộc tính tùy ý như sau:

```js
POST /user/update HTTP/1.1
Host: vulnerable-website.com
...
{
    "user":"wiener",
    "firstName":"Peter",
    "lastName":"Wiener",
    "__proto__":{
        "foo":"bar"
    }
}
```

Nếu trang web có lỗ hổng, thuộc tính được tiêm của bạn sẽ xuất hiện trong đối tượng được cập nhật trong phản hồi:
```js
HTTP/1.1 200 OK
...
{
    "username":"wiener",
    "firstName":"Peter",
    "lastName":"Wiener",
    "foo":"bar"
}
```

Trong một số trường hợp hiếm hoi, trang web thậm chí có thể sử dụng các thuộc tính này để tạo HTML động, dẫn đến việc thuộc tính được đưa vào sẽ được hiển thị trong trình duyệt của bạn.

Khi bạn xác định được khả năng gây prototype pollution phía máy chủ, bạn có thể tìm kiếm các tiện ích tiềm năng để sử dụng cho mục đích khai thác. Bất kỳ tính năng nào liên quan đến việc `cập nhật dữ liệu người dùng` đều đáng để tìm hiểu vì chúng thường liên quan đến việc hợp nhất dữ liệu đầu vào vào một đối tượng hiện có đại diện cho người dùng trong ứng dụng. Nếu bạn có thể thêm các thuộc tính tùy ý cho người dùng của mình, điều này có khả năng dẫn đến một số lỗ hổng, bao gồm cả việc leo thang đặc quyền.

---

## Ví dụ: Privilege escalation via server-side prototype pollution

https://portswigger.net/web-security/prototype-pollution/server-side/lab-privilege-escalation-via-server-side-prototype-pollution

Tìm sink:

Với tính năng update thông tin, ta có thể tiêm prototype pollution vào đây:

![alt text](image-20.png)

Ở đây ta thấy `isAdmin` được set `false` nếu chỉnh sửa bình thường thì không thành công:

![alt text](image-21.png)

Nhưng có thể thành công qua proto:

![alt text](image-22.png)

Cuối cùng chỉ cần vào Admin panel xóa `Carlos`

---

## Phát hiện server-side prototype pollution mà không có được sự phản hồi lại (blind)

Hầu hết thời gian, ngay cả khi bạn làm ô nhiễm thành công một đối tượng nguyên mẫu phía máy chủ, bạn sẽ không thấy thuộc tính bị ảnh hưởng được phản ánh trong phản hồi. Do bạn không thể chỉ kiểm tra đối tượng trong bảng điều khiển nên điều này gây khó khăn khi xác định liệu lệnh tiêm của bạn có hoạt động hay không.

Một cách tiếp cận là thử tiêm các thuộc tính phù hợp với các tùy chọn cấu hình tiềm năng cho máy chủ. Sau đó, bạn có thể so sánh hành vi của máy chủ trước và sau khi tiêm để xem liệu thay đổi cấu hình này có hiệu lực hay không. Nếu vậy, đây là dấu hiệu rõ ràng cho thấy bạn đã tìm thấy thành công lỗ hổng ô nhiễm nguyên mẫu phía máy chủ.

Trong phần này, chúng ta sẽ xem xét các kỹ thuật sau:
- Ghi đè status code
- Ghi đè khoảng trắng JSON
- Ghi đè bộ charset

Tất cả các lần tiêm này đều không phá hủy, nhưng vẫn tạo ra sự thay đổi nhất quán và đặc biệt trong hành vi của máy chủ khi thành công. Bạn có thể sử dụng bất kỳ kỹ thuật nào được đề cập trong phần này để giải quyết bài lab đi kèm.

Đây chỉ là một số ít các kỹ thuật tiềm năng giúp bạn hình dung được những gì có thể thực hiện được. Để biết thêm thông tin chi tiết về kỹ thuật và hiểu biết sâu sắc về cách PortSwigger Research có thể phát triển các kỹ thuật này, hãy xem báo cáo kèm theo có tên `Server-side prototype pollution: Black-box detection without DoS của Gareth Heyes`.

### Status code override

Các framework JavaScript phía máy chủ như `Express` cho phép các nhà phát triển đặt trạng thái phản hồi HTTP tùy chỉnh. Trong trường hợp có lỗi, máy chủ JavaScript có thể đưa ra phản hồi HTTP chung nhưng bao gồm một đối tượng lỗi ở định dạng `JSON` trong nội dung. Đây là một cách cung cấp thêm thông tin chi tiết về lý do xảy ra lỗi, điều này có thể không rõ ràng từ trạng thái HTTP mặc định.

Mặc dù có phần gây hiểu lầm, nhưng việc nhận được phản hồi 200 OK lại khá phổ biến, trong khi nội dung phản hồi lại chứa một đối tượng lỗi có trạng thái khác.

```js
HTTP/1.1 200 OK
...
{
    "error": {
        "success": false,
        "status": 401,
        "message": "You do not have permission to access this resource."
    }
}
```

Mô-đun `http-errors` của Node chứa hàm sau để tạo loại phản hồi lỗi này:

```js
function createError () {
    //...
    if (type === 'object' && arg instanceof Error) {
        err = arg
        status = err.status || err.statusCode || status
    } else if (type === 'number' && i === 0) {
    //...
    if (typeof status !== 'number' ||
    (!statuses.message[status] && (status < 400 || status >= 600))) {
        status = 500
    }
    //...
```

Đầu tiên nó gán biến `status` bằng cách đọc thuộc tính `status` hoặc `statusCode` từ đối tượng được truyền vào hàm. Nếu nhà phát triển trang web chưa thiết lập rõ ràng thuộc tính `status` cho lỗi, bạn có thể sử dụng thuộc tính này để thăm dò ô nhiễm nguyên mẫu như sau:

- Tìm cách kích hoạt phản hồi lỗi và ghi lại mã trạng thái mặc định.
- Hãy thử làm ô nhiễm nguyên mẫu bằng thuộc tính `status` của riêng bạn. Hãy chắc chắn sử dụng status code tối nghĩa mà không có khả năng được cấp vì bất kỳ lý do nào khác.
- Kích hoạt lại phản hồi lỗi và kiểm tra xem bạn đã ghi đè thành công mã trạng thái hay chưa.

Note: Bạn phải chọn mã trạng thái trong phạm vi `400-599`. Nếu không, Node sẽ mặc định là trạng thái 500 bất kể thế nào, như bạn có thể thấy từ dòng được tô sáng thứ hai, do đó bạn sẽ không biết liệu mình có làm ô nhiễm nguyên mẫu hay không.

### JSON spaces override

Framework `Express` cung cấp tùy chọn khoảng trắng `json` cho phép bạn cấu hình số khoảng trắng được sử dụng để thụt lề bất kỳ dữ liệu JSON nào trong phản hồi. Trong nhiều trường hợp, các nhà phát triển để thuộc tính này không xác định vì họ hài lòng với giá trị mặc định, khiến nó dễ bị ô nhiễm thông qua chuỗi nguyên mẫu.

Nếu bạn có quyền truy cập vào bất kỳ loại phản hồi JSON nào, bạn có thể thử làm ô nhiễm nguyên mẫu bằng thuộc tính khoảng trắng json của riêng bạn, sau đó đưa ra lại yêu cầu có liên quan để xem khoảng thụt lề trong JSON có tăng lên tương ứng hay không. Bạn có thể thực hiện các bước tương tự để xóa thụt lề nhằm xác nhận lỗ hổng.

Đây là một kỹ thuật đặc biệt hữu ích vì nó không phụ thuộc vào việc phản ánh một thuộc tính cụ thể nào. Nó cũng cực kỳ an toàn vì bạn có thể bật hoặc tắt chế độ ô nhiễm chỉ bằng cách đặt lại thuộc tính về cùng giá trị mặc định.

Mặc dù lỗi ô nhiễm nguyên mẫu đã được khắc phục trong Express 4.17.4, các trang web chưa nâng cấp vẫn có thể bị tấn công.

Note: Khi thử kỹ thuật này trong Burp, hãy nhớ chuyển sang tab `Raw` của trình soạn thảo tin nhắn. Nếu không, bạn sẽ không thể thấy sự thay đổi thụt lề vì chế độ xem đẹp mắt mặc định sẽ chuẩn hóa điều này.

### Charset override

Máy chủ Express thường triển khai các mô-đun "phần mềm trung gian" cho phép xử lý trước các yêu cầu trước khi chúng được chuyển đến hàm xử lý thích hợp. Ví dụ, mô-đun body-parser thường được sử dụng để phân tích nội dung của các yêu cầu đến nhằm tạo ra đối tượng `req.body`. Phần này chứa một tiện ích khác mà bạn có thể sử dụng để thăm dò ô nhiễm nguyên mẫu phía máy chủ.

Lưu ý rằng đoạn mã sau truyền một đối tượng tùy chọn vào hàm `read()`, được sử dụng để đọc nội dung yêu cầu để phân tích cú pháp. Một trong những tùy chọn này, mã hóa, xác định mã hóa ký tự nào sẽ sử dụng. Mã hóa này có thể được lấy từ chính yêu cầu thông qua lệnh gọi hàm `getCharset(req)` hoặc mặc định là UTF-8.

```js
var charset = getCharset(req) or 'utf-8'

function getCharset (req) {
    try {
        return (contentType.parse(req).parameters.charset || '').toLowerCase()
    } catch (e) {
        return undefined
    }
}

read(req, res, next, parse, debug, {
    encoding: charset,
    inflate: inflate,
    limit: limit,
    verify: verify
})
```

Nếu bạn xem xét kỹ hàm `getCharset()`, có vẻ như các nhà phát triển đã dự đoán rằng tiêu đề Content-Type có thể không chứa thuộc tính charset rõ ràng, vì vậy họ đã triển khai một số logic để chuyển về chuỗi rỗng trong trường hợp này. Điều quan trọng là, điều này có nghĩa là có thể kiểm soát được thông qua ô nhiễm nguyên mẫu.


















