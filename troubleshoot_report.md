# troubleshoot_report.md - Xung đột cổng 8082 (Address already in use)

## 1. Giả lập sự cố
Dừng app để nhường cổng, rồi chạy tiến trình giả chiếm cổng nền:
```bash
sudo systemctl stop spring-app.service
nohup python3 -m http.server 8082 > /tmp/squatter.log 2>&1 &
```

## 2. Chẩn đoán: tìm PID chiếm cổng
```bash
sudo ss -tlnp | grep 8082
# LISTEN 0 5 0.0.0.0:8082 0.0.0.0:* users:(("python3",pid=60271,fd=3))
ps aux | grep http.server | grep -v grep
# chunghoang 60271 ... python3 -m http.server 8082
```
- PID: **60271**, tiến trình: `python3 -m http.server 8082` của user chunghoang.
- Cột cuối của `ss -tlnp` (users:(...pid=...)) cho biết PID.

## 3. Tái hiện lỗi
```bash
sudo systemctl start spring-app.service
sudo journalctl -u spring-app.service --no-pager | grep -i bind
# Exception in thread "main" java.net.BindException: Address already in use
sudo systemctl status spring-app.service
# Active: activating (auto-restart) (Result: exit-code), Main PID exited status=1/FAILURE
```
- Java không bind được 8082 -> crash -> systemd auto-restart theo `Restart=on-failure`.

## 4. Xử lý: giải phóng cổng
```bash
kill 60271
sudo ss -tlnp | grep 8082
# (trống -> cổng đã tự do, kill thường là đủ, chưa cần kill -9)
```

## 5. Khôi phục dịch vụ và xác nhận
```bash
sudo systemctl restart spring-app.service
sudo systemctl status spring-app.service --no-pager
# Active: active (running), Main PID: 60632 (java)
sudo ss -tlnp | grep 8082
# LISTEN ... *:8082 ... users:(("java",pid=60632,fd=5))
curl -s http://127.0.0.1:8082/api/health
# {"status":"UP"}
curl -s http://localhost/api/health
# {"status":"UP"}
```
- Kết luận: cổng 8082 đã chuyển từ `python3` (PID 60271) sang `java` (PID 60632),
  backend và proxy Nginx hoạt động bình thường.
