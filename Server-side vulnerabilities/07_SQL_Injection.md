Đây rồi :|| cái mà tôi mong chờ nhất :||
# SQL Injection là gì
- SQL Injection (SQLi) là một lỗ hổng bảo mật web cho phép atker can thiệp vào truy vấn mà app thực hiện trong dtb. Điều này có thể cho phép atker xem dữ liệu mà bình thường không thể truy cập. Có thể là những data thuộc về user khác, hay những data khác mà app có thể truy cập. Trong nhiều trường hợp, một atker có thể sửa hay xoá data này, làm thay đỏi nội dung/hành vi của app
- Trong một vài tình huống, một atker có thể thực hiện SQL Injection atk vào một server cơ bản, hay các cơ sở hạ tầng back-end. Nó cũng có thể cho phép atker thực hiện tấn công denial-of-service (DOS) - tấn công từ chối dịch vụ

# Làm sao để phát hiện một lỗ hổng SQL Injection
- Bạn có thể phát hiện SQLi thủ công bằng cách sư dụng một test đầu vào của app. Để làm điều này, bạn có thể thử gửi:
  - Một kí tự trích dẫn ```'``` và check lỗi hay sự bất thường nào khác
  - Một số cú pháp SQL cụ thể đánh giá giá trị đầu vào, thử nhập các giá trị rồi đánh giá sự khác nhau giữa các response
  - Điều kiện logic để bypass như ```or 1=1``` hay ```or 1=2```, và kiểm tra sự khác nhau tỏng response
  - Payloads được thiết kế để kích hoạt time delay khi thực thi bên trong một truy vấn SQL, và tìm điểm khác biệt trong response time 
  - OAST payloads được thiết kế để kích hoạt một tương tác mạng ngoài băng tần (out-of-band) khi thực thi truy vấn SQL, và theo dỗi các tương tác phát sinh
- Mặt khác, bạn có thể tìm đa số SQL Injection vuln nhanh gọn lẹ nhờ Burp Scanner :||

# Truy xuất dữ liệu ẩn (Retrieving hidden data)
- Tưởng tượng một app shopping hiển thị sản phẩm ở các danh mục khác nhau. Khi user click vào một danh mục vd nhưu ```Quà tặng```, Browser sẽ request URL:

      https://insecure-website.com/products?category=Gifts
- Điều này khiến app tạo một truy vấn SQL để lấy thông tin sản phẩm từ dtb
  
      SELECT * FROM products WHERE category = 'Gifts' AND released = 1
- Truy vấn SQL này yêu cầu dtb trả về
  - Tất cả thông tin ```*```
  - Từ bảng ```product```
  - ```Category``` là ```gifts```
  - ```released``` là ```1```
- ```released = 1``` là để ẩn sản phẩm mà chưa được release. Ta có thể check các sản phẩm chưa released bằng cách thay bằng 0

# Truy xuất dữ liệu ẩn - Tiếp
- Một app khi không có một biện pháp bảo vệ khỏi SQLi. Thì có nghĩa là atker có thể thực hiện atk, ví dụ:

      https://insecure-website.com/products?category=Gifts'--
- Truy vấn SQL lúc đấy sẽ là

      SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
  - Hãy để ý răng ```--``` là một dấu để viết cmt ở sau nó, đồng nghĩa với việc truy vấn trên sẽ bị bay mất phần ```AND``` ở sau và sẽ trả về mọi sản phẩm cả ẩn và không ẩn
- Ta có thể sử dụng kĩ thuật tương tự để khiến app hiển thị mọi thứ :|| Ví dụ để hiện các danh mục bị ẩn

      https://insecure-website.com/products?category=Gifts'+OR+1=1--
- Khi đó truy vấn sẽ là

      SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
  - Vẫn là bỏ phần ```AND```ở sau nhưng thêm một cái ```OR 1=1```. Khi đấy truy vấn sẽ xét 1 trong 2 điều kiện đúng và 1=1 thì luôn đúng nên coi như truy vấn này sẽ in ra mọi sản phẩm ở mọi category<br>
> [!NOTE]
> Hãy cẩn thận khi inject một điều kiện kiểu ```or 1=1``` vào một truy vấn sql. Ngay cả khi trông nó có vẻ vô hại trong lúc ta inject vào, nhưng việc app sử dụng dữ liệu từ một request đơn trong nhiều truy vấn khác nhau là điều bình thường. Nếu điều kiện vô tình lọt vào cách lệnh ```UPDATE``` hay ```DELETE``` thì đúng là đờ i đi
## Lab văn l
- Vuln: khi user chọn một category, app sẽ thực hiện truy vấn

      SELECT * FROM products WHERE category = 'Gifts' AND released = 1
- Nhiệm vụ là thực hiện SQLi atk :||
- Chọn một Category bất kì, giả sử là ```Pets``` đi, URL của nó trông như này

      https://0a5b00c60413732787321a06004d00e9.web-security-academy.net/filter?category=Pets
- Sửa lại URL để lừa truy vấn :||

      https://0a5b00c60413732787321a06004d00e9.web-security-academy.net/filter?category=Pets'+or+1=1--
- Xong

# Phá vỡ logic app (Subverting App Logic)
- Tưởng tượng một app cho user login với một username và pass. Nếu một user submit username là ```wiener``` và pass là ```bluecheese```, app sẽ kiểm tra thông tin xác thực bằng cách thực hiện truy vấn SQL như sau:

      SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
- Nếu truy vấn này trả về thông tin chi tiết của một user, sau đó login success :||. Không thì là thất bại
- Trong trường hợp này, một atker có thể login như một user bất kì mà không cần pass. Họ có thể chơi mấy cái kiểu nhưu ```--``` để bỏ điều kiện where. Ví dụ, submit username là ```admin'``` và để trống pass sẽ có truy vấn như này

      SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
- Truy vấn trả về user có ```username``` là ```admin``` và login successful
## Lab nóng nảy
- Chứa lỗ hổng SQLi trong login function
- Nhiệm vụ là thực thi SQLi atk để login vào admin
- Vào giao diện login, để username là ```administrator```
