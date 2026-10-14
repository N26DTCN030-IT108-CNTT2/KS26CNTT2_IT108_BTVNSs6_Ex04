# Báo Cáo Bài Tập: Xử Lý Quy Trình Lưu Nhật Ký Hệ Thống (Redirect Operators & Logging)

## Phần 1: So sánh hai toán tử ghi hướng ( > và >> )

| Tiêu chí so sánh | Toán tử Ghi đè (` > `) | Toán tử Ghi nối tiếp (` >> `) |
| :--- | :--- | :--- |
| **Cơ chế hoạt động** | Tạo mới tệp (nếu chưa có) hoặc **ghi đè hoàn toàn** nội dung mới lên tệp cũ. Toàn bộ nội dung cũ sẽ bị xóa sạch. | Tạo mới tệp (nếu chưa có) hoặc **ghi nối tiếp** dữ liệu mới vào cuối tệp hiện có, bảo toàn toàn bộ dữ liệu cũ. |
| **Rủi ro mất dữ liệu** | **Cực kỳ cao.** Nếu vô tình chạy lại lệnh hoặc dùng nhầm vào cuối tháng khi tệp đã tích lũy dữ liệu 30 ngày, toàn bộ báo cáo trước đó sẽ bốc hơi. | **Thấp.** Dữ liệu cũ được giữ nguyên, dữ liệu mới chỉ được nối thêm vào phía sau. |
| **Ngữ cảnh sử dụng thực tế** | Phù hợp khi khởi tạo báo cáo mới tinh, ghi đè tệp log tạm thời, hoặc chạy lệnh reset trạng thái log mỗi khi bắt đầu phiên làm việc mới (Ca sáng). | Phù hợp cho hệ thống giám sát **24/7**, các kịch bản tự động chạy định kỳ liên tục để tích lũy và lưu trữ nhật ký lỗi theo thời gian thực. |
| **Encoding file đầu ra** | Sử dụng mặc định của `Out-File` là **UTF-16** trong PowerShell (nếu dùng toán tử `>`), có thể gây lỗi hiển thị ký tự (mojibake) nếu mở bằng các công cụ chỉ hỗ trợ UTF-8 / ANSI. | Tương tự toán tử `>`, mặc định ánh xạ tới `Out-File` (UTF-16), do đó cần lưu ý khi chọn công cụ đọc file. |

---

## Phần 2: Lựa chọn giải pháp tối ưu cho hệ thống ghi log 24/7

### 1. Giải pháp tối ưu được đề xuất:
Đối với yêu cầu ghi nhật ký lỗi phát sinh liên tục trong ngày và vận hành lâu dài, giải pháp tuyệt đối an toàn và chuyên nghiệp là sử dụng **`Add-Content`** hoặc **`Out-File -Append -Encoding utf8`**.

### 2. Lập luận lý do:
- **Chống mất dữ liệu tuyệt đối:** Cơ chế nối tiếp (`-Append` hoặc `Add-Content`) giúp bảo vệ dữ liệu tích lũy qua nhiều ngày, tránh hoàn toàn thảm họa mất dữ liệu do ghi đè nhầm.
- **Kiểm soát Encoding chuẩn UTF-8:** Mặc định `Out-File` dùng UTF-16 có thể gây bất tiện khi mở trên các trình soạn thảo code (như VS Code). Việc chủ động chỉ định `-Encoding utf8` giúp tệp log tương thích mượt mà trên mọi nền tảng, hiển thị chính xác tiếng Việt có dấu.
- **Cú pháp triển khai chuẩn mực trong Script tự động hóa:**
  ```powershell
  # Sử dụng Add-Content với tường minh mã hóa UTF-8
  Get-Content server.log | Select-String "ERROR" | Add-Content -Path daily_report.txt -Encoding utf8
  
  # Hoặc sử dụng Out-File kết hợp Append và Encoding
  Get-Content server.log | Select-String "ERROR" | Out-File -FilePath daily_report.txt -Append -Encoding utf8
