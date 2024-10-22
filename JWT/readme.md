# JWT
## 1. JWT authentication bypass via unverified signature
https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature

![alt text](image.png)

![alt text](image-1.png)

Context: Không kiểm tra phần chữ kí

![alt text](image-2.png)

---

## 2. JWT authentication bypass via flawed signature verification
https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification

![alt text](image-3.png)

![alt text](image-4.png)

Thay đổi `alg` thành `none`:\
![alt text](image-5.png)

---

## 3. JWT authentication bypass via weak signing key
https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key

Context: Secret key dễ đoán 

Brute-force: `hashcat -a 0 -m 16500 <YOUR-JWT> /path/to/jwt.secrets.list`

![alt text](image-6.png)

![alt text](image-7.png)

---

## 4. JWT authentication bypass via jwk header injection
https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jwk-header-injection


















