# HƯỚNG DẪN CẤU HÌNH VÀ VẬN HÀNH CI/CD TRÊN GITHUB ACTIONS & VPS

Dự án đã được tích hợp toàn bộ quy trình CI/CD vào trong **1 file duy nhất**:
👉 [`.github/workflows/ci_cd.yml`](.github/workflows/ci_cd.yml)

Pipeline gồm 2 job liên kết tuần tự:
1. **Job `ci`**: Kiểm thử (Unit Test + Integration Test), đóng gói file JAR và kiểm tra build Docker image.
2. **Job `cd`**: Tự động SSH vào VPS để kéo mã nguồn mới nhất và chạy lại hệ thống qua Docker Compose sau khi `ci` hoàn tất.

---

## 1. Cơ chế hoạt động của Pipeline

- **Khi có Pull Request**: Chỉ chạy giai đoạn `ci` để kiểm tra code (không kích hoạt `cd` để tránh deploy nhầm code chưa duyệt).
- **Khi push vào nhánh `main` hoặc `master`**: Tự động chạy tuần tự `ci` ➔ thành công ➔ tự động chạy `cd` lên VPS.
- **Kích hoạt thủ công**: Có thể bấm **Run workflow** trên tab Actions bất cứ lúc nào.

---

## 2. Các thông tin VPS cần điền (Trong ngoặc nhọn `{...}`)

Trong file [`.github/workflows/ci_cd.yml`](.github/workflows/ci_cd.yml), các trường kết nối VPS gồm:

| Placeholder cần điền | Ý nghĩa | Ví dụ mẫu |
| :--- | :--- | :--- |
| `{IP_VPS_CUA_BAN}` | Địa chỉ IP Public của VPS | `103.179.188.45` |
| `{USER_VPS_CUA_BAN}` | Tên đăng nhập SSH trên VPS | `root` hoặc `ubuntu` |
| `{PASSWORD_VPS_CUA_BAN}` | Mật khẩu đăng nhập SSH | `MatKhau@123` |
| `{SSH_PRIVATE_KEY_VPS}` | Khóa riêng tư SSH (nếu dùng key thay mật khẩu) | `-----BEGIN OPENSSH PRIVATE KEY...` |
| `{PORT_SSH_THUONG_LA_22}` | Cổng kết nối SSH (mặc định là 22) | `22` |
| `{THU_MUC_DU_AN_TREN_VPS}` | Thư mục đặt dự án trên VPS | `/root/department-service` hoặc `/home/ubuntu/app` |

---

## 3. Cách cấu hình an toàn qua GitHub Secrets (Khuyên dùng)

Vào repository trên GitHub:
1. Chọn **Settings** -> **Secrets and variables** -> **Actions**.
2. Nhấn **New repository secret** và thêm các biến tương ứng:
   - `VPS_HOST`: Điền `{IP_VPS_CUA_BAN}`
   - `VPS_USERNAME`: Điền `{USER_VPS_CUA_BAN}`
   - `VPS_PASSWORD`: Điền `{PASSWORD_VPS_CUA_BAN}` (hoặc `VPS_SSH_KEY` nếu dùng SSH key)
   - `VPS_PORT`: `22`
3. Trong file `ci_cd.yml`, cập nhật đường dẫn `{THU_MUC_DU_AN_TREN_VPS}` theo thư mục bạn muốn chứa code trên VPS.

*(Nếu muốn cấu hình nhanh không qua Secrets, bạn có thể điền thông tin trực tiếp thay thế các chuỗi `{...}` trong file `ci_cd.yml`).*

---

## 4. Chuẩn bị trên VPS trước khi chạy lần đầu

Đăng nhập vào VPS và chạy lệnh:
```bash
# Cập nhật package và cài Docker, Docker Compose, Git
sudo apt update && sudo apt install -y docker.io docker-compose git

# Cho phép user chạy docker không cần sudo (nếu dùng user non-root như ubuntu)
sudo usermod -aG docker $USER
sudo systemctl enable --now docker
```

---

## 5. Kiểm tra kết quả sau khi Deploy
- Swagger UI tài liệu API: `http://{IP_VPS_CUA_BAN}:8081/swagger-ui.html`
- Kiểm tra container trên VPS: `docker compose ps`
