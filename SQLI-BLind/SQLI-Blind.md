
# SQL Injection Blind in DVWA

Mức độ: **LOW**, **MEDIUM**, **HIGH**

---

## 🔹 LEVEL: LOW

### 📌 Description:
<img width="606" height="172" alt="image" src="https://github.com/user-attachments/assets/3f185261-fd87-4695-9cf3-8597293109c7" />


Lab cung cấp một trường input để nhập `ID` của người dùng, và kết quả trả về dựa trên input đó.  

<img width="601" height="148" alt="image" src="https://github.com/user-attachments/assets/eccb5d7d-ac49-4dc4-8fd7-4f04c794a5c5" />

Tuy nhiên, kết quả chỉ trả về thông báo:

- “**User ID exists in the database.**”  
- hoặc “**User ID is MISSING from the database.**”

→ Không trả về dữ liệu thực trong database.

---

### 🎯 Goals:
- Trích xuất thêm nhiều thông tin hơn từ cơ sở dữ liệu backend.

---

### 🔍 Phân tích:

- Do kết quả trả về không hiển thị dữ liệu cụ thể, ta phải khai thác dựa trên logic **true/false** hiển thị.
- Đây là kỹ thuật **Boolean-based Blind SQLi**.

---

#### ✅ Source Code:

```sql
SELECT first_name, last_name FROM users WHERE user_id = (:id) LIMIT 1;
```

- Biến `ID` không được lọc → dễ dàng chèn payload SQL.

---

### 🧪 Kỹ thuật khai thác:

<img width="603" height="112" alt="image" src="https://github.com/user-attachments/assets/3f472834-2b4b-432a-b846-3421e7c0a2a8" />

Thiết kế mệnh đề truy vấn để dò tên bảng trong database:

```sql
1' AND SUBSTRING((SELECT table_name FROM information_schema.tables WHERE table_schema = database() LIMIT 1 OFFSET 0),{position},1) = '{char}'
```

Trong đó:

- `{position}`: từ 1 đến 20 trong vòng lặp
- `{char}`: từng ký tự trong bảng chữ cái và số

➡️ Payload này dò từng ký tự của tên bảng bằng cách kiểm tra xem truy vấn trả về là **"exists"** hay **"missing"**

> 💬 *P/S: Tool này em tự viết nhé sếp ơi :vv*

---

### ✅ Kết quả:

- Tên bảng đầu tiên thu được là: `users`
<img width="605" height="57" alt="image" src="https://github.com/user-attachments/assets/6d354585-83af-4f81-bf15-c261b7e2023d" />

- Muốn tìm bảng tiếp theo → tăng `OFFSET` trong câu query
<img width="594" height="186" alt="image" src="https://github.com/user-attachments/assets/69671b0b-90b3-48bf-a8d1-9ded9e2ed990" />


---

### 🔍 Tìm tên các cột trong bảng `users`:

<img width="606" height="46" alt="image" src="https://github.com/user-attachments/assets/547e215f-1e06-4ab9-8547-031e33c9a5ac" />

```sql
1' AND SUBSTRING((SELECT column_name FROM information_schema.columns WHERE table_name = 'users' LIMIT 1 OFFSET 0),{position},1) = '{char}'#
```

> ⚠️ Lưu ý: phải encode payload này khi truyền vào URL thì mới hoạt động chính xác.

- Kết quả:
<img width="597" height="225" alt="image" src="https://github.com/user-attachments/assets/2a41e5fa-a42a-4aee-a5a2-0f9235c2fadb" />

---

### ✅ Kết luận:
- Có thể lặp lại kỹ thuật trên để lấy tên nhiều cột khác trong bảng `users`.
- Đây là kỹ thuật Blind SQL Injection thông qua phân tích phản hồi **logic true/false**.

