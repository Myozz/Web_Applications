# Server-side request forgery (SSRF) - Giả mạo request phía server là gì?
- SSRF là một lỗ hổng bảo mật web mà cho phép một atker khiến app phía server thực hiện request tới một vị trí không mong muốn
- Trong một SSRF atk thông thường, atker thường sẽ khiến server thực hiện kết nối tới một service nội bộ của một cơ sở hạ thông thuộc một tổ chức nào đấy. Trong trường hợp khác, họ có thể ép server kết nối tới hệ thống bên ngoài tuỳ ý. Điều này có thể leak các dữ liệu nhạy cảm, như là thông tin uỷ quyền

# SSRF atk against the server
- Trong một SSRF atk chống lại server, atker khiên app tạo một http request trở lại server chủ của chính nó, thông qua giao diễn loopback network của nó. Điều này thường liên quan tới việc cung cấp URL với hostname kiểu ```127.0.0.1``` (một địa chỉ IP dành riêng trỏ tới loopback adapter)
- Ví dụ, tưởng tượng một app shopping mà cho phép user xem một mặt hàng có sẵn hàng hay không. Để cung cập thông tin này, app phải thực hiện truy vấn tới các back-end REST API khác nhau. Nó làm điều này bằng cách vượt qua URL tới back-end API endpoint có liên quan thông qua một front-end HTTP request. Khi một user xem trạng thái hàng, browser của họ sẽ tạo một request:

      POST /product/stock HTTP/1.0
      Content-Type: application/x-www-form-urlencoded
      Content-Length: 118
      stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1  

- Điều này khiến server thực hiện request tới một URL cụ thể, để lấy thông tin hàng, và gửi lại cho user
- Trong ví dụ này, một atker có thể sửa đổi request tới một local URL cụ thể tới server

      POST /product/stock HTTP/1.0
      Content-Type: application/x-www-form-urlencoded
      Content-Length: 118
      
      stockApi=http://localhost/admin

- Server sẽ tìm nạp nội dung của ```/admin``` và gửi lại cho user, thế là đi'
- Atker có thể truy cập vào ```/admin``` URL, nhưng đương nhiên không sử dụng được các tính năng cho admin vì không được uỷ quyền. Điều này có nhiều một atker sẽ không thấy gì đáng giá đâu. Tuy nhiên, nếu request tới ```/admin``` URL từ local machine (máy của user mực tiêu), thế là bypass được kiểm soát truy cập thông thường. App sẽ có toàn quyền truy cập vào các tính năng của admin, bởi vì request đến từ vị trí được tin tưởng (trursted location) - Túm lại khiến máy của user request đến localhost, khi đấy nó sẽ tự truy cập được vào do thấy request đến từ nguồn tin cậy
