# Bài 5: Chẩn đoán và xử lý xung đột cổng (Address Already in Use)

Báo cáo chi tiết xem file `troubleshoot_report.md` trong repo này.

Tóm tắt:
1. Giả lập: `nohup python3 -m http.server 8082 &` (PID 60271).
2. Chẩn đoán: `sudo ss -tlnp | grep 8082` -> `users:(("python3",pid=60271,fd=3))`.
3. Lỗi tái hiện: `java.net.BindException: Address already in use`, service `activating (auto-restart)`.
4. Xử lý: `kill 60271` (kill thường là đủ).
5. Khôi phục: `sudo systemctl restart spring-app.service` -> `active (running)`,
   `*:8082` do `java` (PID 60632) giữ, `/api/health` trả `{"status":"UP"}` cả trực tiếp và qua Nginx.
