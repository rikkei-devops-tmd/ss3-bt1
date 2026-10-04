BÀI TẬP 1: THAY ĐỔI CỔNG KẾT NỐI SSH (SSH PORT HARDENING)

1. MỤC TIÊU
- Thay đổi cổng dịch vụ SSH mặc định từ 22 sang 2222 trên máy chủ Ubuntu 22.04 LTS nhằm giảm thiểu nguy cơ bị rà quét tự động và tấn công Brute-Force.
- Thiết lập tường lửa UFW (Uncomplicated Firewall) mở cổng mới 2222/tcp trước khi khởi động lại dịch vụ SSH daemon để chống rủi ro mất quyền truy cập (Lockout).
- Thực hiện toàn bộ quy trình bằng tài khoản người dùng thường devops có quyền sudo, tuân thủ nguyên tắc đặc quyền tối thiểu.

2. CÁC BƯỚC THỰC HIỆN CHI TIẾT

2.1. Đăng nhập vào máy chủ bằng tài khoản devops
Thực hiện kết nối SSH vào máy chủ bằng user devops qua cổng 22 hiện tại:
```bash
ssh -i ~/.ssh/id_ed25519_do devops@159.223.45.67
```

2.2. Mở cổng 2222/tcp trên tường lửa UFW
Trước khi thay đổi cấu hình SSH, bắt buộc mở cổng 2222 trên UFW để tránh bị ngắt kết nối:
```bash
sudo ufw allow 2222/tcp comment 'Custom SSH Port'
sudo ufw status verbose
```

2.3. Chỉnh sửa tệp cấu hình SSH Daemon (/etc/ssh/sshd_config)
Mở tệp cấu hình SSH daemon bằng trình soạn thảo:
```bash
sudo nano /etc/ssh/sshd_config
```
Tìm dòng cấu hình cổng và thay đổi từ Port 22 sang Port 2222:
```text
Port 2222
```
Lưu tệp và kiểm tra tính hợp lệ của tệp cấu hình trước khi khởi động lại dịch vụ:
```bash
sudo sshd -t
```

2.4. Khởi động lại dịch vụ SSH và kiểm tra trạng thái
Áp dụng cấu hình mới bằng lệnh:
```bash
sudo systemctl restart ssh
sudo systemctl status ssh
```
Kiểm tra tiến trình SSH đang lắng nghe trên cổng 2222:
```bash
sudo ss -tulpn | grep sshd
```

2.5. Xóa quyền truy cập cổng 22 trên UFW
Sau khi đã chắc chắn cổng 2222 hoạt động ổn định, tiến hành đóng cổng 22:
```bash
sudo ufw delete allow 22/tcp
sudo ufw reload
sudo ufw status verbose
```

3. KẾT QUẢ KIỂM TRA VÀ LOG TERMINAL

3.1. Kiểm tra trạng thái cổng lắng nghe và tường lửa trên máy chủ
```text
devops@ptit-devops-server:~$ sudo ss -tulpn | grep ssh
tcp   LISTEN 0      128          0.0.0.0:2222      0.0.0.0:*    users:(("sshd",pid=1420,fd=3))
tcp   LISTEN 0      128             [::]:2222         [::]:*    users:(("sshd",pid=1420,fd=4))

devops@ptit-devops-server:~$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
2222/tcp                   ALLOW IN    Anywhere                   # Custom SSH Port
80/tcp                     ALLOW IN    Anywhere                  
443/tcp                    ALLOW IN    Anywhere                  
2222/tcp (v6)              ALLOW IN    Anywhere (v6)              # Custom SSH Port
80/tcp (v6)                ALLOW IN    Anywhere (v6)             
443/tcp (v6)               ALLOW IN    Anywhere (v6)             
```

3.2. Kiểm tra kết nối từ máy tính cá nhân qua cổng mới 2222 (Thành công)
```text
$ ssh -p 2222 -i ~/.ssh/id_ed25519_do devops@159.223.45.67
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-88-generic x86_64)

Last login: Sun Oct  4 10:15:32 2026 from 14.162.180.25
devops@ptit-devops-server:~$ whoami
devops
devops@ptit-devops-server:~$ echo $SSH_CONNECTION
14.162.180.25 54231 159.223.45.67 2222
```

3.3. Kiểm tra kết nối từ máy tính cá nhân qua cổng cũ 22 (Bị chặn)
```text
$ ssh -p 22 -i ~/.ssh/id_ed25519_do devops@159.223.45.67
ssh: connect to host 159.223.45.67 port 22: Connection refused
```

4. KẾT LUẬN
- Dịch vụ SSH đã chuyển đổi thành công sang cổng 2222.
- Tường lửa UFW đã cấu hình bảo vệ toàn diện, ngăn chặn hoàn toàn kết nối vào cổng mặc định 22.
- Tài khoản quản trị devops thực thi đầy đủ quyền hạn an toàn thông qua sudo.
