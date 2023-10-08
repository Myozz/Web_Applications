# Server-side request forgery (SSRF) - Giả mạo request phía server là gì?
- SSRF là một lỗ hổng bảo mật web mà cho phép một atker khiến app phía server thực hiện request tới một vị trí không mong muốn
- Trong một SSRF atk thông thường, atker thường sẽ khiến server thực hiện kết nối tới một service nội bộ của một cơ sở hạ thông thuộc một tổ chức nào đấy. Trong trường hợp khác, họ có thể ép server kết nối tới hệ thống bên ngoài tuỳ ý. Điều này có thể leak các dữ liệu nhạy cảm, như là thông tin uỷ quyền

# SSRF atk against the server
- Trong một SSRF atk chống lại server, atker khiên app tạo một http request trở lại server chủ của chính nó, thông qua giao diễn loopback network của nó. Điều này thường liên quan tới việc cung cấp URL với hostname kiểu ```127.0.0.1``` (một địa chỉ IP dành riêng trỏ tới loopback adapter)
- 
