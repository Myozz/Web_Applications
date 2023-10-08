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

# SSRF atk against the server - part 2
- Tại sao app lại hoạt động theo cách này, và ngầm tin tưởng request đến từ local machine? Điều này có thể nảy sinh từ các lí do sau:
  - Kiểm tra kiểm soát truy cập có thể được triển khai trong một đối tượng khác năm trước app server. Khi một kết nối được thiết lập trở lại server, sẽ bypass được đợt kiểm tra
  - Với mục đích khắc phục thiệt hai, app có thể cho phép admin truy cập mà không cần login, tới mọi user từ các local machine. Điều này cung cấp một cách cho admin để khôi phục hệ thống nếu họ mất thông tin xác thực của họ. Điều này giả định rằng chỉ người dùng hoàn toàn được tin tưởng mới đến trực tiếp từ máy chủ
  - Giao diện admin có thể nghe một port khác tới main app, và có thể không thể truy cập trực tiếp bởi user
- Những kiểu trust này, nơi mà request đến từ local machine được xử lý khác với request từ bên ngoài, thường khiến SSRF trở thành một lỗ hổng nghiêm trọng
## Lab
- Đống lý thuyết bên trên dịch ra thực sự lú não vcl
- Ok thì lab này có một feature check hàng hoá bằng cách tìm nạp dữ liệu từ hệ thống nội bộ
- Để giải lab này, thay đổi URL kiểm tra để truy cập vào giao diện admin tại ```http://localhost/admin``` và xoá user ```carlos```
- Trước hết, vào xem một sản phẩm bất kỳ ở trang chủ lab, sau đó bật intercept ở tab proxy
- Check stock ![image](https://github.com/Myozz/Web_Applications/assets/94811005/d56d66c9-69cf-4af4-94d6-bd65aacc682b)
- Để ý dòng StockAPI ![image](https://github.com/Myozz/Web_Applications/assets/94811005/3aaa6ca9-123a-4fc5-8b2d-72c01cdfd13f)
- Chuột phải, send to repeater rồi đổi stockapi thành ```http://localhost/amdin``` rồi send
![image](https://github.com/Myozz/Web_Applications/assets/94811005/c726b6ce-b705-463a-be18-144e332773f3)
- Tìm xem dòng path để delete carlos ở đâu, lấy nó rồi quay lại Proxy để thay stockapi bằng path lấy được rồi tiến hành forward ![image](https://github.com/Myozz/Web_Applications/assets/94811005/789e71d7-175c-48c2-9868-9bfbd7677391)

# SSRF atk against other Back-end Sys (SSRF tấn công chống lại hệ thống back-end khác)
- Phụ đạo thêm một chút về back-end thì nó là phần cốt lõi của một website, mang chức năng lưu trữ dữ liệu, xử lý các yêu cầu của người dùng, Database là một ví dụ dễ thấy nhất của Back-end
- Trong một số trường hợp, app server có thể tương tác với back-end sys mà không thể truy cập trực tiếp bởi user. Những hệ thống này thường có những IP riêng tư mà không thể định tuyến được. Back-end sys thường được bảo vệ bởi cấu trúc liên kết mạng (network topology), vậy nên bảo mật của chúng thường yếu hơn. Trong đa số trường hợp, back-end sys nội bộ sẽ chứa những tính năng nhạy cảm mà có thể được truy cập mà không cần xác thực bởi bất kỳ ai có thể tương tác với hệ thống
- Ở ví dụ trước, tưởng tượng có một giao diện admin ở back-end URL ```https://192.168.0.68/admin```(thay vì localhost như ví dụ trước) Một atker có thể gửi những request sau để khai thác lỗ hổng SSRF, và truy cập vào giao diện adin

        POST /product/stock HTTP/1.0
        Content-Type: application/x-www-form-urlencoded
        Content-Length: 118
        stockApi=http://192.168.0.68/admin
