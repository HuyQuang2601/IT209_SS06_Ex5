# Báo cáo Bài 5 - Systemd Service cho ứng dụng Java

## 1. Nội dung thực hiện

- Tạo tài khoản hệ thống `java-runner` không có quyền đăng nhập shell.
- Cài đặt unit file tại `/etc/systemd/system/java-app.service`.
- Chạy dịch vụ dưới quyền `java-runner`.
- Dùng `/usr/bin/sleep infinity` để mô phỏng ứng dụng Java chạy dài hạn.
- Tự khởi động lại sau 10 giây khi tiến trình thoát do lỗi.

## 2. Cài đặt

Thực hiện trên máy Linux có Systemd, từ thư mục chứa hai tệp bài nộp:

```bash
sudo useradd --system --user-group --no-create-home --shell /usr/sbin/nologin java-runner
sudo install -o root -g root -m 0644 java-app.service /etc/systemd/system/java-app.service
sudo systemctl daemon-reload
sudo systemctl enable --now java-app.service
```

Nếu user đã tồn tại, thông báo `useradd: user 'java-runner' already exists` có thể bỏ qua.

## 3. Kiểm tra trạng thái

```bash
sudo systemctl status java-app.service --no-pager
```

Kết quả mong đợi (PID, thời gian và tên máy sẽ khác):

```text
● java-app.service - Java Application Service (simulation)
     Loaded: loaded (/etc/systemd/system/java-app.service; enabled; preset: enabled)
     Active: active (running)
   Main PID: 1234 (sleep)
      Tasks: 1
     Memory: ...
        CPU: ...
     CGroup: /system.slice/java-app.service
             └─1234 /usr/bin/sleep infinity
```

## 4. Kiểm tra nhật ký và user thực thi

```bash
sudo journalctl -u java-app.service -n 20 --no-pager
sudo systemctl show java-app.service -p User -p Restart -p RestartUSec
```

Kết quả nhật ký mong đợi có các dòng:

```text
java-app initialized successfully
uid=... (java-runner) gid=... (java-runner) groups=... (java-runner)
```

Kết quả cấu hình mong đợi:

```text
User=java-runner
Restart=on-failure
RestartUSec=10s
```

## 5. Kiểm tra cơ chế tự khởi động lại

Gửi tín hiệu kết thúc bất thường cho tiến trình chính:

```bash
sudo systemctl kill --signal=SIGKILL --kill-who=main java-app.service
sudo systemctl status java-app.service --no-pager
sudo journalctl -u java-app.service -n 30 --no-pager
```

Sau khoảng 10 giây, dịch vụ phải trở lại trạng thái `active (running)`. Nhật ký sẽ cho thấy tiến trình cũ bị lỗi và Systemd đã lên lịch rồi khởi động lại dịch vụ.

## 6. Ghi chú kết quả

Tệp báo cáo này được chuẩn bị trong môi trường Windows nên không thể chạy `systemctl` để thu kết quả thực tế. Khi thực hành trên máy Linux, thay phần kết quả mẫu ở mục 3 bằng đầu ra thực tế của lệnh `systemctl status java-app.service --no-pager` trước khi nộp bài.
