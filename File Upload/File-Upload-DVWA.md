
# File Upload - DVWA Writeup

Mức độ: **LOW**, **MEDIUM**, **HIGH**

---

## 🔹 LEVEL: LOW

### 📌 Description:
<img width="606" height="257" alt="image" src="https://github.com/user-attachments/assets/79e605c7-a91f-4504-86b9-61d981819b68" />

Lab cho phép upload 1 file ảnh lên server và có thể truy cập trực tiếp file ảnh đó sau khi upload.
<img width="608" height="232" alt="image" src="https://github.com/user-attachments/assets/c1f8b98a-becf-44a5-8f82-8d919aeb1ef1" />

---

### 🎯 Goals:
Upload một webshell lên server và khiến server **thực thi** file đó.

---

### 🔍 Phân tích:

- Web cho phép upload file mà không có kiểm tra gì cả.
- Ta có thể upload 1 file có nội dung độc hại (ví dụ shell PHP) thay vì file ảnh bình thường.
- Vì backend viết bằng PHP, shell cũng phải là PHP thì server mới thực thi được.

---

### 🧪 Thực hành:

Upload file `test.php` với nội dung:

```php
<?php system('ls /'); ?>
```

<img width="542" height="73" alt="image" src="https://github.com/user-attachments/assets/ad0a0e0f-1ca0-4a8b-b5e9-836ddcf8366c" />


- Web không có bất kỳ lớp kiểm tra nào → shell được upload thành công.

<img width="598" height="209" alt="image" src="https://github.com/user-attachments/assets/fcaa6dd7-5508-4c3e-ad9d-1d425b85836d" />

- Truy cập đường dẫn tới file đã upload → **server thực thi shell.**
<img width="609" height="88" alt="image" src="https://github.com/user-attachments/assets/db930777-8699-42ac-b1f0-eb9cc0e2e59e" />

---

## 🔸 LEVEL: MEDIUM

### 🔍 Phân tích:

- Ở level này, web **kiểm tra thêm các thuộc tính của file**: `type`, `size`, `name`.
- Chỉ cho upload nếu `Content-Type` là `image/jpeg` hoặc `image/png`.

> Vậy làm sao upload shell PHP?

### 💡 Cách bypass:

- **Giữ định dạng file là `.php`**
- Dùng **Burp Suite** để chỉnh thủ công `Content-Type` trong gói tin thành `image/jpeg`

→ Bypass được kiểm tra `Content-Type`.

---

### 🧪 Thực hành:

- Upload file `.php`, chỉnh `Content-Type` trong request thành `image/jpeg`

<img width="631" height="442" alt="image" src="https://github.com/user-attachments/assets/ee3d89ea-a617-4ca1-81f1-22a0ab73afd9" />

- Kết quả: Bypass thành công → server thực thi shell

---

## 🔺 LEVEL: HIGH

### 🔍 Phân tích:

- Lớp lọc nâng cấp hơn so với level trước.

#### 🧩 Kiểm tra phần mở rộng:

```php
$uploaded_ext = substr($uploaded_name, strrpos($uploaded_name, '.') + 1);
```

- Dùng kỹ thuật `.png.php` để bypass sẽ **không hiệu quả** ở đây nữa.

#### 🧩 Kiểm tra nội dung ảnh:

```php
getimagesize($uploaded_tmp);
```

- Hàm này sẽ **kiểm tra phần đầu file** (header) xem có đúng là ảnh hay không.
- Nếu bạn rename file `.php` thành `.jpg`, hoặc nhúng shell vào ảnh nhưng nội dung không đúng chuẩn ảnh → vẫn bị chặn.

---

### 💡 Bypass nâng cao:

- Chèn shell PHP vào **cuối nội dung file ảnh**
- Vì `getimagesize()` **chỉ check phần đầu file**, nên shell nằm cuối sẽ không bị phát hiện.

---

### 🧪 Thực hành:

- Upload file ảnh `.jpg` thật

<img width="610" height="137" alt="image" src="https://github.com/user-attachments/assets/9820e75a-9637-43f5-b69d-cb8da0693472" />

- Dùng tool chèn đoạn PHP shell vào **cuối file**

<img width="614" height="204" alt="image" src="https://github.com/user-attachments/assets/51617024-f267-433b-80b9-094c8eff1da9" />

> File vẫn được upload thành công vì vượt qua được cả phần kiểm tra extension và content-type.

---

### ⚠️ Vấn đề:

- Khi truy cập file ảnh chứa shell → **server chỉ hiển thị ảnh**, không thực thi shell.

<img width="614" height="248" alt="image" src="https://github.com/user-attachments/assets/abea76cd-9ee2-43ff-bedf-53370452abe8" />

---

### 🧠 Giải pháp:

Tận dụng lỗ hổng **File Inclusion** (LFI/RFI) để ép server **thực thi nội dung ảnh có shell bên trong**.

---

### 🎁 Bonus: Về File Inclusion

- Đây là lỗi cho phép attacker chỉ định đường dẫn file để server `include()` hoặc `require()` file đó.

Ví dụ:

```php
include('shell.jpg');
```

- Nếu attacker đã upload file `.jpg` chứa shell PHP → server thực thi nội dung trong ảnh.

---

### ✅ Kết quả:
<img width="616" height="215" alt="image" src="https://github.com/user-attachments/assets/30d74a8e-b9a2-41dd-9e0a-818e8baa6146" />

- Tận dụng lab File Inclusion, trỏ tới file `.jpg` chứa shell
- Server **thực thi** shell nằm bên trong file ảnh → khai thác thành công.

