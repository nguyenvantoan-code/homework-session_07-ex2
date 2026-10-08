# Bài 2: Quản trị Tường lửa UFW cho Cụm Dịch vụ Multi-port

## Mục tiêu

- Thiết lập tường lửa UFW để bảo mật hệ thống máy chủ chạy nhiều cổng dịch vụ đồng thời.
- Hiểu cách cấu hình mở rộng cổng cho các cổng dịch vụ web và cô lập các cổng cơ sở dữ liệu.

---

## Bối cảnh

Máy chủ chạy 3 dịch vụ:

| Dịch vụ       | Cổng      | Yêu cầu                                      |
|---------------|-----------|----------------------------------------------|
| SSH           | 22/tcp    | Cho phép từ bên ngoài (quản trị)             |
| Nginx (HTTP)  | 80/tcp    | Cho phép từ bên ngoài (web tiêu chuẩn)       |
| Spring Boot   | 8082/tcp  | Cho phép từ bên ngoài (kiểm tra từ xa)       |
| MySQL         | 3306/tcp  | **Chặn hoàn toàn** (không thêm luật allow)  |

---

## Các lệnh cấu hình UFW

```bash
# 1. Khai báo chính sách mặc định
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 2. Cho phép các cổng dịch vụ
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 8082/tcp

# 3. Kích hoạt UFW
sudo ufw enable
```

---

## Kết quả kiểm tra

**Lệnh kiểm tra:**

```bash
sudo ufw status verbose
```

**Đầu ra:**

```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
8082/tcp                   ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
80/tcp (v6)                ALLOW IN    Anywhere (v6)
8082/tcp (v6)              ALLOW IN    Anywhere (v6)
```

---

## Giải thích kết quả

| Tiêu chí kiểm tra                                     | Kết quả |
|-------------------------------------------------------|---------|
| `Status: active` — UFW đang hoạt động                | ✅ Đạt  |
| `22/tcp` có trạng thái `ALLOW IN` từ Anywhere        | ✅ Đạt  |
| `80/tcp` có trạng thái `ALLOW IN` từ Anywhere        | ✅ Đạt  |
| `8082/tcp` có trạng thái `ALLOW IN` từ Anywhere      | ✅ Đạt  |
| `3306` **không xuất hiện** trong danh sách (bị chặn) | ✅ Đạt  |

---

## Giải thích bảo mật

- **`default deny incoming`**: Chặn tất cả lưu lượng đầu vào theo mặc định — đây là nguyên tắc *least privilege*, chỉ mở những gì thực sự cần thiết.
- **`default allow outgoing`**: Cho phép máy chủ tự do kết nối ra ngoài (cập nhật package, gọi API, v.v.).
- **Cổng MySQL 3306 không được thêm luật allow**: Cơ sở dữ liệu chỉ nên truy cập nội bộ (qua `localhost` / mạng nội bộ). Việc để 3306 lộ ra Internet là rủi ro bảo mật nghiêm trọng, dễ bị tấn công brute-force hoặc khai thác lỗ hổng.
- **Spring Boot 8082**: Chỉ mở trong môi trường phát triển/kiểm thử; trong production nên đặt sau reverse proxy (Nginx) và đóng cổng này lại.
