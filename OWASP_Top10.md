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

# Broken Authentication
- Các lỗ hổng trong hệ thống xác thực (login) có thể cho phép atker truy cập vào user's acc và thậm chí có thể play luôn toàn bộ hệ thống bằng tài khoản admin/root gì đấy.
- VÍ dụ: atker có thể lấy một danh sách chứa hàng nghìn tổ hợp username/passwd (chắc giống wordlist hoặc lấy từ những lần atker trước) và sử dụng lệnh để thử tất cả lên hệ thống mục tiêu rồi cầu may
- Nghe quen vc đúng không :|| nó rõ ràng là Brute-force còn gì (sai thì thôi), ngoài brute-force (mò pass), ngoài ra còn 2 hình thức phổ biến khác:
  - Use of weak credentials (vẫn là mò pass nhưng nó thủ công): Nếu một hệ thống để cho user đặt mật khẩu quá dễ đoán thì hacker chả cần làm gì nhiều :|| mò phát ra luôn (như kiểu mấy thằng cu set admin:admin)
  - Weak session Cookies: Cơ bản là chôm cookies nhét vào máy mình (hình như có cả tự chế nữa :|| không rõ lắm) để đánh lừa hệ thống đấy là máy của thằng chủ đống cookies bị chôm
- Một số cách để giảm thiểu lỗ hổng này:
  - 2FA: Xác thực 2 yếu tố, cái này rõ ai cũng biết rồi
  - Giới hạn số lần đăng nhập: Khi đăng nhập sai một số lần nhất định, sẽ ngăn đăng nhập tiếp. Đương nhiên những cu hackẻ lỏd thì vẫn có cách để fake HWID, NetID và đủ thứ khác nếu cần để đánh lừa hệ thống đấy là người khác, như thế có thể tiếp tục đăng nhập (tuỳ hệ thống thì nó còn chặn đăng nhập trên toàn hệ thống, nhưng thế thì lại khổ thằng user thật)
  - Thời gian tạm dừng giữa các lần đăng nhập sai: Như kiểu đt sai tầm 5 lần là bắt đợi 5 phút rồi tăng dần, hay tk ngân hàng nhập sai nhiều lần là coi như đi. Cái này tác động trực tiếp vào tài khoản của user đấy nên là có fake như nào, thâm chí cả user thực sự của nó cũng ăn block :|| bảo mật thật nhưng chỉ khổ thằng bị tấn công
- **Ví dụ thú vị**:
  - Tôi thấy có cái bài này hay cực trên THM :||
