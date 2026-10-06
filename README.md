## 1. Mục tiêu

Tạo nhóm `devops-admin`, tài khoản `deployer`, và cấp cho nhóm quyền chạy `systemctl` (start, stop, restart, status) không cần mật khẩu thông qua `/etc/sudoers`, không cấp toàn quyền root.

## 2. Nhật ký các lệnh đã thực hiện

### Bước 1: Tạo nhóm và người dùng

```bash
sudo groupadd devops-admin
sudo adduser deployer
sudo usermod -aG devops-admin deployer
id deployer
```

Kết quả `id deployer`:

```text
<dán đầu ra>
```

### Bước 2: Cấu hình sudoers bằng visudo

```bash
sudo visudo
```

Dòng cấu hình đã thêm vào cuối file `/etc/sudoers`:

```text
%devops-admin ALL=(ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

Xác nhận rule đã được lưu:

```bash
sudo grep devops-admin /etc/sudoers
```

Giải thích: dấu `%` biểu thị một nhóm; `ALL=(ALL)` cho phép chạy trên mọi máy với danh nghĩa mọi user; `NOPASSWD:` bỏ yêu cầu nhập mật khẩu; chỉ các lệnh `systemctl` được liệt kê mới được phép. Dùng `visudo` thay vì sửa trực tiếp vì nó kiểm tra cú pháp trước khi lưu, tránh làm hỏng sudo.

## 3. Kiểm tra

```bash
su - deployer
sudo -l
sudo systemctl restart cron
exit
```

Kết quả `sudo -l` khi đăng nhập bằng `deployer`:

```text
User deployer may run the following commands on DESKTOP-OFRUK8L:
    (ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

Kết quả `sudo systemctl restart cron`: lệnh chạy thành công, không yêu cầu nhập mật khẩu của `deployer` và không có thông báo lỗi.

## 4. Đối chiếu kết quả

| Yêu cầu                                                  | Kết quả |
|----------------------------------------------------------|---------|
| `sudo -l` hiển thị NOPASSWD systemctl cho %devops-admin  | [PASS]  |
| `sudo systemctl restart cron` không hỏi mật khẩu         | [PASS]  |

## 5. Nhận xét an toàn

Ký tự `*` khớp mọi tham số phía sau nên quy tắc này rộng hơn cần thiết (ví dụ cho phép cả `systemctl restart ssh`). Trong thực tế nên liệt kê từng dịch vụ cụ thể, chẳng hạn `/usr/bin/systemctl restart cron`, để tuân thủ nguyên tắc đặc quyền tối thiểu.

## 6. Kết luận

Đã tạo nhóm, người dùng và cấu hình sudoers đúng yêu cầu; `deployer` chỉ quản lý được dịch vụ, không có toàn quyền root.