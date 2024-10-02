# Insecure deserialization

### 1. Modifying serialized objects
https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects

Context: Trường admin được đặt trong cookie và dùng giá trị boolean để xác định.

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```
![alt text](image.png)

Sửa cookie thành `O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}` rồi encode base64 và gửi đi ta có thể thấy xuất hiện admin panel

---

### 2. Modifying serialized data types
https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-data-types

Context: Xuất hiện `access-token` unique cho từng tài khoản, ta sẽ đổi giá trị này thành int và có giá trị là 0.

![alt text](image-1.png)

![alt text](image-2.png)

---

### 3. Using application functionality to exploit insecure deserialization
https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-using-application-functionality-to-exploit-insecure-deserialization

Context: Bài này ở đoạn serialize có chứa 1 object lưu lại path đến avatar, ta sẽ thao túng path này để xóa nội dung. 

Khi truy cập `/delete` thì sẽ có đoạn:\
`s:11:"avatar_link";s:23:"path..."`\
Nhằm xóa luôn avatar khi xóa tài khoản. Vì vậy nếu đây là 1 file nhạy cảm khác nó sẽ xóa luôn trên server. 

Payload:\
`s:11:"avatar_link";s:23:"/home/carlos/morale.txt"`

---

### 4. Arbitrary object injection in PHP - Tiên đối tượng tùy ý
https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-arbitrary-object-injection-in-php

Context: 
