# Authentication Vulnerabilites (Lỗ hổng xác thực)
- Về mặt khái niệm, lỗ hổng xác thực khá dễ hiểu. Tuy nhiên, chúng rất quan trong bởi mối quan hệ giữa xác thực và bảo mật
- Lỗ hổng xác thực có thể cho phép atker chiếm quyền truy cập vào các dữ liệu và tính năng nhạy cảm. Bởi lí do này, rất quan trọng để học cách xác định và khai thác lỗ hổng xác thực, và làm thế nào để bypas các biện pháp bảo vệ thông thường
- Ở phần này, chúng ta sẽ nghiên:
  - Cơ chế xác thực phổ biến nhất được sử dụng bởi web
  - Những lỗ hổng tiềm năng của các cơ chế này
  - Những lỗ hổng vốn có trong các cơ chế xác thực khác nhau
  - Những lỗ hổng đặc trưng do việc triển khai không đúng cách (config đần)
  - Làm thế nào để tự tạo một cơ chế xác thực ngon nhất có thể
  - 
# Điểm khác biệt giữa xác thực và uỷ quyền (Authentication & Authorization)
- Xác thực là một tiến trình xác minh một user là chính họ (tức là người login có phải chính chủ không). Còn uỷ quyền thì là xác minh xem user đó có quyền để thực hiện một chức năng nào đó không
- Ví dụ, xác thực xác định liệu ai đó đang cố gắng truy cập vào một web với username ```Carlos123``` có thực sự là người dùng đã tạo tài khoản này không
- Một khi ```Carlos123``` được xác thực, quyền của họ quyết định xem họ được uỷ quyền những gì. Ví dụ, họ có thể được uỷ quyền để truy cập vào thông tin cá nhân của các user khác, hay thực hiện hành đồng như xoá user's account

# Brute-force atk
- Một brute-force atk là khi một atker sử dụng một hệ thống thử nghiệm và test lỗi để đoán ra các tên người dùng đang tồn tại. Kiểu tấn công này thông thường được thực hiện tự động với wordlist các username và password. Tự động hoá tiến trình này, đặc biệt là sử dụng các công cụ chuyên dụng, sẽ cho ra khả năng thử login ở tốc độ cao
- Brute-force không phải lúc nào cũng là đoàn username và pass một cách hoàn toàn random. Bằng cách sử dụng logic và những thông tin có sẵn, atker có thể tinh chỉnh brute-force để khiến việc đoán tốt hơn. Nó cải thiện dáng kể độ hiệu quả của đợt tấn công.
- Website mà chỉ dựa vào login bằng pass thông thường mà không có thêm phương thức nào khác để xác thực user có thể sẽ dễ dàng bị khai thác nếu không thực hiện các biện pháp chống brute-force

# Brute-forcing usernamé
- Username cực kì dễ đoàn nếu chúng tuân thủ một mô hình dễ nhận biết, như là một email address. Ví dụ, ta thường thấy mọi người login theo format này ```firstname.lastname@somecompany.com```. Tuy nhiên, thậm chí không có một hình mẫu rõ ràng, thỉnh thoảng tài khoản đặc quyền cao cũng dùng những từ dễ đoàn, như là ```admin``` hay ```administrator```
- Trong suốt quá trình theo dõi, kiểm tra nếu website tiết lộ username một cách công khai. Ví dụ, bạn có thể truy cập vào tài khoản mà không cần login không? Thâm chí nội dung thực tế của profile bị ẩn, tên được sử dụng trong profile đôi khi lại giống username.Bạn cũng nên kiểm tra HTTP response để xem email/username nào đang tồn tại. Thỉnh thoảng, response bao gồm email với user đặc quyền cao, như là admin hay IT support
