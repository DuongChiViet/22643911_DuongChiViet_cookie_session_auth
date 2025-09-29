# COOKIE_SESSION_AUTH — Xác thực bằng Cookie + Session (Express + MongoDB)

Hệ thống xác thực người dùng bằng Cookie + Session sử dụng `express-session` và `MongoDB` (lưu session với `connect-mongo`).

Repo: `22643911_DuongChiViet_cookie_session_auth`

## 📑 Mục lục

- ⚙️ Cài đặt & Chạy dự án
- 📝 CÂU B — REGISTER
	- 🆕 Đăng ký tài khoản
	- 🗄️ Kiểm tra trong Database
	- 🚫 Truy cập khi chưa đăng nhập
- 🔑 Đăng nhập
- 📝 CÂU D — Profile
- 📝 CÂU E — Logout
- 🔗 API endpoints tóm tắt
- 🧩 Ghi chú & Khắc phục lỗi

---

## ⚙️ Cài đặt & Chạy dự án

Yêu cầu:
- Node.js >= 18
- MongoDB đang chạy cục bộ tại `mongodb://127.0.0.1:27017`

Thư mục chính có các file chính:
- `app.js` — khởi động server, cấu hình session + MongoStore
- `routes/auth.js` — các endpoint auth: register, login, profile, logout
- `models/User.js` — schema `User` + hash mật khẩu bằng `bcryptjs`

Các bước:

1) Cài dependencies

```powershell
npm install
```

2) Chạy dự án

```powershell
node app.js
```

Mặc định server chạy tại: http://localhost:3000 và sử dụng DB: `sessionAuth`.

---

## 📝 CÂU B — REGISTER

### 🆕 Đăng ký tài khoản

Dùng Postman → `POST /auth/register`

Body (JSON):

```json
{
	"username": "admin",
	"password": "12345"
}
```

Kết quả (200): `{"message":"User registered successfully!"}`

### 🗄️ Kiểm tra trong Database

Sau khi đăng ký, tài khoản được lưu vào collection `users` trong DB `sessionAuth`.

### 🚫 Truy cập khi chưa đăng nhập

Gọi `GET /auth/profile` khi CHƯA login:

URL: `http://localhost:3000/auth/profile`

Kết quả: `401 Unauthorized`

Lý do có thể:
- Chưa đăng nhập
- DB chưa có thông tin xác thực hợp lệ
- Client chưa gửi cookie `connect.sid` (ssid) lên server

---

## 🔑 Đăng nhập

`POST /auth/login`

Body (JSON) — dùng đúng thông tin đã đăng ký ở trên:

```json
{
	"username": "admin",
	"password": "12345"
}
```

Kết quả mong đợi:
- ✅ Đăng nhập thành công (`{"message":"Login successful!"}`)
- ✅ Server trả về cookie phiên `connect.sid` (tự sinh bởi `express-session`)
- ✅ Session được lưu trong MongoDB (collection mặc định: `sessions`)

---

## 📝 CÂU D — Profile

Sau khi đăng nhập thành công, gọi `GET /auth/profile`:

- Nếu có session hợp lệ (cookie `connect.sid` gửi kèm), server sẽ đọc `req.session.userId` và trả về thông tin user (đã ẩn trường `password`).
- Nếu không, trả về `401 Unauthorized`.

---

## 📝 CÂU E — Logout

`GET /auth/logout`

Kết quả mong đợi:
- ✅ Session trên server bị hủy (`req.session.destroy`)
- ✅ Cookie `connect.sid` trên client được xóa (`res.clearCookie('connect.sid')`)
- ✅ Bản ghi session trong DB cũng bị xóa

---

## 🔗 API endpoints tóm tắt

- `POST /auth/register` — Đăng ký tài khoản mới
	- Body: `{ "username": string, "password": string }`
- `POST /auth/login` — Đăng nhập, set cookie `connect.sid`
	- Body: `{ "username": string, "password": string }`
- `GET /auth/profile` — Lấy thông tin người dùng (YÊU CẦU đã đăng nhập)
- `GET /auth/logout` — Đăng xuất, hủy session + xóa cookie

---

## 🧩 Ghi chú & Khắc phục lỗi

### Cấu hình phiên (trong `app.js`)
- `cookie.httpOnly: true`: chặn JS client-side đọc cookie.
- `cookie.secure: false`: đặt `true` khi deploy HTTPS.
- `maxAge: 1h`: thời hạn cookie 1 giờ.
- `store: connect-mongo`: lưu session vào MongoDB để scale/out và tránh mất session khi restart.

### Postman không giữ cookie?
- Bật “Automatically follow redirects” nếu dùng redirect (ở đây không dùng, nhưng nên bật mặc định).
- Sau khi gọi `/auth/login`, kiểm tra tab “Cookies” trong Postman có `connect.sid`.
- Khi gọi `/auth/profile`, đảm bảo request gửi kèm cookie.

### 401 Unauthorized khi gọi `/auth/profile`
- Chưa gửi cookie `connect.sid` (client không lưu hoặc không gửi lại).
- Session đã hết hạn (`maxAge`), cần login lại.
- Server bị restart và store session không persistent (trong repo này đã dùng Mongo, OK).

### Không kết nối được MongoDB
- Kiểm tra MongoDB đã chạy: `mongodb://127.0.0.1:27017`.
- Kiểm tra DB name: `sessionAuth`.
- Port bị bận hoặc service chưa start.

### Trùng username khi đăng ký
- Schema đặt `unique: true` cho `username` ⇒ Nếu đăng ký trùng sẽ lỗi. Dùng username khác hoặc xóa record cũ.

---

## 🔍 Tham khảo nhanh cấu trúc code

- `app.js`
	- Kết nối MongoDB
	- Cấu hình `express-session` + `connect-mongo`
	- Mount route `/auth`
- `routes/auth.js`
	- `POST /auth/register` — tạo user (hash mật khẩu tự động ở model)
	- `POST /auth/login` — xác thực; set `req.session.userId`
	- `GET /auth/profile` — trả user đang đăng nhập (nếu có session)
	- `GET /auth/logout` — hủy session + xóa cookie
- `models/User.js`
	- Hash mật khẩu bằng `bcryptjs` (`pre('save')`)
	- So sánh mật khẩu bằng `comparePassword`

---

✨ Hoàn tất! Hệ thống xác thực bằng Cookie + Session đã hoạt động đầy đủ:

- 🆕 Đăng ký
- 🔑 Đăng nhập
- 👤 Xem profile (chỉ khi login)
- 🚪 Đăng xuất

🚀 Ready to use!