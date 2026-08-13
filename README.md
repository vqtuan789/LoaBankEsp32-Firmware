# LoaBank2026 Firmware

Repository public này chứa **firmware chính thức và hướng dẫn cập nhật** cho thiết bị LoaBank2026.

> Mã nguồn phát triển được quản lý riêng. Repository này không chứa API Key SePay, mật khẩu Wi-Fi hoặc dữ liệu cá nhân của người dùng.

## Phiên bản

Xem mục **Releases** để tải các firmware đã phát hành.

Tên firmware ESP32 tiêu chuẩn:

```text
LoaBank2026-esp32.bin
```

Các bản ESP32-S3 sẽ được phát hành riêng khi target ESP32-S3 hoàn tất kiểm thử.

## Cập nhật firmware qua WebServer

Thiết bị hỗ trợ OTA qua GitHub.

### 1. Mở WebServer

Điện thoại/máy tính phải ở cùng mạng LAN với LoaBank.

Mở:

```text
http://<IP-cua-LoaBank>:8386
```

Ví dụ nếu thiết bị đang có IP `192.168.1.88`:

```text
http://192.168.1.88:8386
```

### 2. Xác thực trang cấu hình

Nhập mã quản trị của thiết bị.

Từ firmware V2.1.9, sau khi xác thực thành công, WebServer sẽ **tự kiểm tra phiên bản mới một lần** nếu GitHub OTA Manifest URL đã được cấu hình.

Thiết bị **không tự cài firmware**. Người dùng vẫn phải chủ động bấm nút cập nhật.

### 3. Kiểm tra trạng thái firmware

WebServer hiển thị một trong các trạng thái:

```text
Firmware đang là bản mới nhất
```

hoặc:

```text
Có firmware mới Vx.y.z
```

Nếu có phiên bản mới, bấm **Cập nhật firmware** và chờ thiết bị hoàn tất.

### 4. Trong lúc OTA

Không:

- rút nguồn ESP32;
- tắt modem/router;
- reset thiết bị;
- cố phát lại giao dịch liên tục.

Firmware được tải vào OTA app slot, kiểm tra đúng kích thước và SHA-256 trước khi hoàn tất cập nhật.

### 5. Sau OTA

Thiết bị tự khởi động lại.

Các cấu hình cá nhân nằm trong NVS nên bình thường vẫn được giữ nguyên:

- SSID và mật khẩu Wi-Fi;
- DHCP / IP tĩnh;
- IP / Gateway / Subnet / DNS;
- API Key SePay;
- chế độ Polling / Webhook / Both;
- ECountdown;
- lời chào khởi động;
- GitHub OTA Manifest URL.

Sau khi thiết bị lên lại, mở WebServer và kiểm tra version mới.

## Nếu OTA không thành công

Nếu WebServer báo lỗi kiểm tra/cập nhật:

1. Không reset cấu hình và không Erase Flash.
2. Kiểm tra Internet của modem/router.
3. Chờ loa phát âm thanh xong rồi thử lại.
4. Bấm **Kiểm tra OTA** lại.
5. Nếu vẫn lỗi, ghi lại thông báo trên Web và Serial log nếu có.

Nếu thiết bị vẫn chạy firmware cũ thì dữ liệu người dùng chưa bị mất; OTA thất bại sẽ không chủ động xóa NVS.

## Cấu hình Wi-Fi khi đổi modem/router

Giữ nút BOOT để vào Config Mode theo firmware hiện tại, kết nối Access Point cấu hình của LoaBank và nhập lại Wi-Fi.

Nếu thiết bị đã có SSID/password nhưng router chỉ đang khởi động chậm sau mất điện, LoaBank sẽ **không tự chuyển vào Config Mode**. Thiết bị tiếp tục thử kết nối SSID đã lưu và chờ Internet phục hồi.

## ECountdown

ECountdown là hạn mức do người dùng cấu hình.

Giá trị hiển thị:

```text
ECountdown cấu hình - tổng số giao dịch từ đầu tháng
```

Nếu số giao dịch vượt hạn mức, OLED/Web có thể hiển thị số âm. Giá trị cấu hình gốc không bị ghi giảm theo runtime.

Nếu để ECountdown trống, chức năng này được tắt và không hiển thị trên OLED.

## Chế độ nhận giao dịch

Firmware hỗ trợ:

- **Polling**: ESP32 chủ động hỏi SePay khoảng 6 giây/lần, lấy 3 giao dịch gần nhất.
- **Webhook**: SePay gửi giao dịch về ESP32 qua endpoint webhook đã cấu hình/NAT.
- **Both**: sử dụng cả hai; transaction ID/reference chung được dùng để chống thông báo trùng.

## Phát lại giao dịch gần nhất

Có thể dùng nút **Phát lại giao dịch gần nhất** trên WebServer hoặc nút BOOT theo chức năng của firmware.

Firmware có cooldown để tránh người dùng bấm liên tục gây áp lực RAM/TLS.

## Khi mất điện hoặc mất Internet

LoaBank được thiết kế để tự phục hồi:

```text
Có điện lại
-> ESP32 khởi động
-> router chưa sẵn sàng: tiếp tục chờ/retry
-> Wi-Fi có lại: tự kết nối
-> Internet chưa có: Web local vẫn hoạt động, SePay chờ
-> Internet phục hồi: SePay tự resync giao dịch + monthly count
```

Không cần vào Config Mode chỉ vì modem khởi động chậm.

## Sửa lỗi nhỏ trước khi báo lỗi

Nếu loa không cập nhật giao dịch hoặc Web báo Internet/SePay lỗi tạm thời:

1. Chờ 30-60 giây xem hệ thống tự phục hồi.
2. Kiểm tra Wi-Fi/Internet trên WebServer.
3. Không bấm Replay liên tục trong lúc mạng đang hồi phục.
4. Nếu lỗi kéo dài, restart modem trước và để LoaBank tự reconnect.
5. Chỉ restart LoaBank nếu hệ thống không tự phục hồi sau modem đã ổn định.
6. Khi báo lỗi, gửi kèm version firmware và đoạn log quanh thời điểm xảy ra lỗi.

## Release và tính toàn vẹn firmware

Mỗi release nên có:

```text
LoaBank2026-esp32.bin
SHA-256
Release notes
```

OTA manifest public có cấu trúc:

```json
{
  "version": "2.1.9",
  "esp32": {
    "url": "https://github.com/vqtuan789/LoaBankEsp32-Firmware/releases/download/v2.1.9/LoaBank2026-esp32.bin",
    "size": 0,
    "sha256": ""
  },
  "esp32s3": {
    "url": "",
    "size": 0,
    "sha256": ""
  }
}
```

`size` và `sha256` phải được điền bằng giá trị thực của binary đã phát hành.

## Lưu ý bảo mật

- Không đăng API Key SePay, password Wi-Fi hoặc `config` của khách hàng vào Issues/Discussions.
- Không tải firmware từ nguồn không thuộc repository/release chính thức.
- Nếu cần gửi log để sửa lỗi, kiểm tra và che credential trước khi đăng công khai.

## Hỗ trợ

Khi cần hỗ trợ, cung cấp tối thiểu:

- version firmware;
- loại board ESP32;
- hiện tượng xảy ra;
- khoảng thời gian lỗi;
- log Serial quanh thời điểm lỗi;
- thao tác đã thực hiện trước khi lỗi xuất hiện.
