- Giới thiệu qua thì OWASP TOP 10 là danh sách 10 lỗ hổng bảo mật web quan trọng cũng như phổ biến nhất. Danh sách này được OWASP (Open Web App Security Project) duy trì để giúp cải thiện bảo mật các ứng dụng web
- Danh sách này dựa nhận OWASP TOP 10 của năm 2021, tôi tôi có hỏi AI về top 10 và nó có đưa cho tôi một danh sách của năm 2023 nhưng vì không tìm được trên trang chủ nên tạm thời tôi sẽ dùng danh sách 2021

# A01: Broken Access Control (Vi phạm kiểm soát truy cập)
- BAC đề cập đến các lỗ hổng bảo mật cho phép người dùng trái phép atker truy cập vào dữ liệu (data) hay chức năng (function) mà họ không hề có quyền truy cập trước đó. Dẫn tới việc tiết lộ thông tin trái phép, sửa đổi/phá huỷ dữ liệu, hay thực hiện các chức năng nằm ngoài quyền hạn người dùng
- Các lỗ hổng phổ biến:
  - Sửa đổi trong URL
  - Sửa đổi thông tin nhận dạng để truy cập tài khoản người khác (IDOR - Insecure direct object references) Insecure direct object references
  - Leo thang đặc quyền
- Các ngăn chặn:
  - Từ chối mọi truy cập vào mọi tài nguyên trừ các tài nguyên được công khai
  - Xác thực người dùng khi họ quay lại ứng dụng
  - Kiểm tra quyền hạn cùa người dùng tại thời điểm người dùng cố gắng thực hiện hàng động
- VD: Atker có thể buộc trình duyệt đến các URL mục tiêu, thường là các trang quản trị. Ví dụ: https://example.com/app/admin

# A02: Cryptographic Failures (Lỗi mã hoá)
- Bảo mật thông tin bằng cách mã hoá thông tin theo các cách khác nhau, nhưng nếu cách mã hoá đó có thể được giải mã hay là cách thức giải mã không đảm bảo an toàn thì sẽ gây rò rỉ thông tin
- Các lỗi phổ biến:
  - Sử dụng giao thức truyền dữ liệu dạng rõ như HTTP, FTP,...
  - Sử dụng những mã hoá đã lỗi thời
  - Khoá bí mật quá dễ đoán
  - Chuỗi mã hoá không được xác thực
- Cách ngăn chặn
  - Không sử dụng những giao thức cũ như FTP, SMTP,... để vận chuyển dữ liệu nhạy cảm
  - Đảm bảo các thuật toán mã hoá đạt tiêu chuẩn mạnh mã
  - Mã hoá dữ liệu trển đường truyền TLS, HTTPS
  - Lưu trữ pass bằng cách hàm hash mạnh
- Ví dụ: 
# A03: Injection
- Các lỗ hổng injection xảy ra khi mã độc được chèn vào ứng dụng thông qua các trường đầu vào của người dùng (biểu mẫu, url,...). Mã này sau đó có thể được chính ứng dụng thực thi, dẫn đến việc dữ liệu bị đánh cắp, truy cập trái phép hay thậm chí là xâm nhập hệ thống
- Một số loại injection phổ biến:
  - SQLi: Chèn vào một đoạn mã vào trong csdl để thực hiện truy vấn dữ liệu
  - Cmd Inj: Chèn những câu lệnh hệ thống độc hại (thường ở dạng tập lệnh) để khai thác dữ liệu nhạy cảm
  - XSS (Cross-site Scripting): Chèn những đoạn mã vào trang web mục đích để khai thác dữ liệu trên trang web
- Ngăn chặn:
  - Sử dụng API an toàn, tránh sử dụng hoàn toàn trình thông dịch, cộng chuỗi trong truy vấn
  - Lọc đầu vào truy vấn, thoát các kí tự đặc biệt
- Ví dụ: ![image](https://github.com/Myozz/Web_Applications/assets/94811005/70ed5722-09d5-440c-b25a-1690243913e1)

# A04: Insecure Design (Thiết kế không an toàn)
- Lỗ hổng này đề cập đến các lỗi cơ bản trong thiết kế của ứng dụng khiến ứng dụng dễ bị tấn công. Điều này có thể bao gồm những thứ như không sử dụng các các mã hoá an toàn, không xác thực đầu vào của user đúng cách hoặc không lưu trữ dữ liệu một cách an toàn
- Cách ngăn chặn:
  - Thiết lập sử dụng những thư viện mẫu thiết kế an toàn
  - Kiểm tra tính hợp lí ở mỗi cấp ứng dụng
  - Tách các lớp phần trên hệ thống và các lớp mạng
  - Hạn chế tiêu thụ tài nguyên người dùng hoặc dịch vụ
- Ví dụ: Một rạp chiếu phim cho phép đặt chỗ theo nhóm tối đa 15 người trước khi đặt tiền cọc, một kẻ tấn công có thể chạy lệnh để đặt tất cả các chỗ trong rạp sau đó dừng lại ở bước đặt cọc, gây tổn thất lớn về kinh tế
  
# A05: Security Misconfiguretion (Cấu hình bảo mật sai)
- Ngay cả với các tính năng bảo mật mạnh mẽ, nếu không được cấu hình đúng cách thì chúng có thể bị vô hiệu hoá. Điều này có thể liên quan tới việc cấu hình sai web server, csdl hay phần mềm bảo mật
- Các lỗi phổ biến:
  - Các tính năng không cầnthiết được bật
  - Thiếu việc tăng cường bảo mật cho từng phần của ứng dụng
  - Các tài khoản và mật khẩu được để mặc định
  - Phần mềm lỗi thời
- Ngăn chặn:
  - Loại bỏ những tài nguyên, tính năng không cần thiết
  - Cung cấp sự hiệu quả và an toàn giữa các thành phần
  - Liên tục cập nhật các phần mềm lên phiên bản mới nhất
- Ví dụ: Danh sách thư mục không bị tắt trên máy chủ. Kẻ tấn công phát hiện ra chúng có thể liệt kê các thư mục một cách đơn giản. Điều này có thể dẫn đến kẻ tấn công dịch ngược lại đoạn code và là tiềm ẩn rất lớn cho nhiều mối nguy hiểm khác

# A06: Vulnerable and Outdated Components (Các thành phần dễ bị tổn thương và lỗi thời)
- Việc sử dụng các thành phần phần mềm dễ bị tôn thương và lỗi thời, chẳng hạn như thư viện, framework hay plugin, có thể đưa vào các rủi ro bảo mật vào ứng dụng của bạn. Các thành phần này có thể có các lỗ hổng đã biết mà kẻ tấn công có thể khai thác để truy cập hệ thống của bạn
- Những lỗi phổ biến:
  - Không quét lỗ hổng thường xuyên và đăng ký nhận các bản tin bảo mật liên quan đến các thành phần mà ta đang sử dụng
  - Phần mềm dễ bị tấn công, không được hỗ trợ hoặc lỗi thời
  - Không sửa chữa, nângcấp các nền tảng
  - Không bảo mật cấu hình của các thành phần
- Ngăn chặn:
  - Loại bỏ các phụ thuộc không sử dụng, các tính năng, thành phần không cần thiết
  - Liên tục kiểm tra các phiên bản của cả thành phần phía máy khách và máy chủ
- Ví dụ: Các thành phần thường chạy với các đặc quyền giống như chính ứng dụng đó, vì vậy sai sót trong bất kỳ thành phần nào có thể dẫn đến tác động nghiêm trọng. Những sai sót như vậy có thể là ngẫu nhiên hoặc cố ý

# A07: Indentification and Authentication Failure (Nhận dạng và xác thực thất bại)
- Các lỗi thường gặp phải
  - Không có biện pháp ngăn chặn brute force
  - Cho phép đặt mật khẩu yếu
  - Xác thực đa yếu tố không hiệu quả
  - Đường dẫn "Quên mật khẩu" được thiết lập kém an toàn
- Ngăn chặn
  - Giới hạn số lần xác thực
  - Kiểm tra độ mạnh của mật khẩu
  - Đảm bảo đường dẫn khôi phục tài khoản được tăng cường để chống lại các cuộc tấn công
- Ví dụ: Bạn đặt mật khẩu quá dễ đoán hay là ứng dụng không giới hạn số lần đăng nhập và kẻ tấn công thực hiện cuộc tấn công từ điển

# A08: Software and Data Integrity Failure (Lỗi toàn vẹn dữ liệu và phần mềm)
- Các lỗi về tính toàn vẹn của phần mềm và dữ liệu liên quan đến code và cơ sở hạ tầng không bảo vệ khỏi các vi phạm tính toàn vẹn
  - Ứng dụng dựa vào các plugin, thư viện hoặc module không đáng tin cậy, không an toàn. Dẫn tới truy cập trái phép, thực thi các mã độc hại hoặc xâm nhập hệ thống
  - Tự động cập nhật các bản cập nhật mà không xác minh tính toàn vẹn đầy đủ và được áp dụng cho phiên bản trước đó
- Ngăn chặn:
  - Sử dụng chữ kí số để xác minh phần mềm
  - Đảm bảo các thưu viện và phần phụ thuộc
  - Có công cụ bảo mật để kiểm tra độ an toàn phần mềm
  - Đảm bảo những dữ liệu chưa được kí hoặc chưa được mãh oá không gửi đến các máy khách không đáng tin cậy
- Ví dụ: Cập nhật mà không cần kí, người dùng sẽ vô tình tải về những bản cập nhật chứa mã độc mà kẻ tấn công cố tình phát tán trên mạng để đánh cắp thông tin hay khai thác dữ liệu trong máy nạn nhân

# A09: Security Logging and Monitoring Failures (Các lỗi theo dõi và ghi nhật kí bảo mật)
- Ghi nhật kí bảo mật nhằm giúp phát hiện, báo cáo và phản hồi các vị phạm nhằm kịp thời ngăn chặn các cuộc tấn công nguy hiệm. Ghi nhật kí giám sát và phản hồi không đầy đủ có thể xảy ra bất cứ lúc nào
  - Các sự kiện quan trọng như đăng nhập không thành công hay những thao tác có tác động lớn không được ghi lại
  - Các cảnh báo lỗi không thông báo, không đầy đủ hoặc không rõ ràng
  - Nhật kí các hoạt động API không được giám sát
  - Ứng dụng không thể hoặc phản hồi quá chậm các phát hiện, báo cáo hoặc cảnh cáo về cách cuộc tấn công đang hoạt động trong thời gian thực
- Ngăn chặn:
  - Đảm bảo các lỗi đăng nhập, kiểm soát truy cập và xác thực đầu vào phía máy chủ được ghi lại đầy đủ để xác định các tài khoản đáng ngờ
  - Đảm bảo nhật kí được mã hoá chính xác, tránh việc hệ thống ghi nhật kí hoặc giám sát bị tấn công 

# Server-side Request Forgecy (SSRF - Giả mạo yêu cầu phía máy chủ)
- SSRF xảy ra bất cứ khi nào khi mà web app đang tìm nạp tài nguyên từ xa mà không xác thực URL do người dùng cung cấp. Nó cho phép atker ép web app gửi một request đến một điểm đích không mong muốn, ngay cả khi được bảo vệ bởi tường lửa
- Ngăn chặn
  - Lớp mạng:
    - Phận chia đoạn chức năng truy cập tài nguyên từ xa trong các mạng riêng biệt để giảm tác động của SSRF
    - Thực thi các chính sách tường lửa "từ chối theo mặc định" hoặc các quy tắc kiểm soát
  - Lớp ứng dụng
    - Làm sạch và xác thực tất cả dữ liệu đầu vào do người dùng cung cấp
    - Thực thi lược đồ URL, cổng là điểm đến với dánh sách cho phép xác thực
    - Tắt chuyển hướng HTTP
- Ví dụ: Những kẻ tấn công có thể truy cập các tệp cục bộ chẳng hạn như hoặc các dịch vụ nội bộ để lấy thông tin nhạy cảm như ```file:///etc/passwd``` và ```http://localhost:28017/admin```
