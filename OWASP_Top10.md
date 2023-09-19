# Giới thiệu :||
- OWASP (Open Web Application Security Project) là một tổ chức phi lợi nhuận tập trung vào việc tìm hiểu các công nghệ và cách khai thác web, đồng thời cung cấp các tài nguyên và công cụ được thiết kế để cải thiện tính bảo mật của software app
- Top 10 bao gồm
  - Injection
  - Broken Authentication
  - Sensitive Data Exposure
  - XML External Entity
  - Broken Access Control
  - Security Misconfiguration
  - Cross-site Scripting
  - Insecure Deserialization
  - Components with Known Vulnerabilities
  - Insufficent Logging & Monitoring
# Injection (Điền form lừa code)
- Injection attack xảy ra khi dữ liệu không đáng tin cậy được gửi đến trình thông dịch mã (code interpreter) thông qua việc việc điền các form (biểu mẫu) hoặc một số dữ liệu khác gửi đến ứng dụng web. Một số ví dụ phổ biến như:
  - SQL Injection: Xảy ra khi input của user được truyển tới các truy vấn SQL. Và attacker có thể chuyển các truy vấn SQL để thao túng kết quả của chúng
  - Command Injection: Xảy ra khi user input chuyển tới sys cmd. Và atker có thể thực thi lệnh hệ thống tuỳ ý trên app server
- Có thể được ngăn chặn bằng cách sau:
  - Dùng các danh sách cho phép (Using an allow list): Khi input được gửi tới server, input này sẽ được so sánh với một danh sach input hay char an toàn. Nếu được cho là an toàn thì nó mới được xử lý. Nếu không thì nó bị từ chối và app sẽ trả về lỗi
  - Khử trùng input (stripping input): Nếu input chứa những char đáng ngờ, chúng sẽ bị loại bỏ trước khi đem đi xử lý
## OS cmd Injection
- Cmd Injection xảy ra khi server-side code (ví dụ php) trong web app thực hiện sys call trên máy chủ. Nó là một lỗ hổng web cho phép atker lợi dụng sys call để thực thi OS cmd trên server
- Thỉnh thoảng không phải là do ác ý, chỉ vào whoami hay đọc file. Nhưng vấn đề của cmd injection là nó mở ra quá nhiều tuỳ chọn cho atker. Thứ tệ nhất họ có thể làm là triệu hồi một reverse shell :|| để bú root hoặc đại khái thế. Một câu lệnh đơn giản như ```;nc -e /bin/bash``` là tất cả những gì cần để chiếm quyền làm bố server (đương nhiên là còn cả tỉ con đường reverse shell khác nữa)
