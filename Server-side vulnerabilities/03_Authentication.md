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

# Brute-forcing password
- Pass có thể brute-force tương tự, với độ khó phụ thuộc vào độ mạnh của pass. Nhiều website áp dụng một số dạng chính sách pass, điều này yêu cầu user để tạo nên pass có độ phức tạp cao, ít nhất thì sẽ gây khó khăn khi chỉ dùng brute-force để tấn công. Điều này thường yêu cầu pass có những đặc điểm sau:
  - Số kí tự ít nhất
  - Kết hợp kí tự hoa và thường
  - Ít nhất một kí tự đặc biệt
- Tuy nhiên, trong khi mật khẩu phức tạp khó cho máy tính crack một mình, ta có thể sử dụng những kiến thức cơ bản về hành vi con người để khai thác lỗ hổng mà user vô tình làm lộ trong hệ thống. Thay vì tạo một pass mạnh với những kí tự ngẫu nhiên, user thường dùng pass dễ nhớ và cố gắng làm đó phù hợp với chính sách pass của hệ thống. Ví dụ, nếu ```mypassword``` không được cho phép, thì họ sẽ dùng kiểu ```Mypassword!``` hay ```Myp4$$w0rd```
- Sự hiểu biết về thông tin xác thực có khả năng và mô hình dễ đoán có nghĩa là brute-force atk có thể thươngf phức tạp hơn rất nhiều, và vì vậy rất hiệu quả, hơn là thử bừa mọi thứ có thể

# Liệt kê username
- Liệt kê username là khi một atker có thể quan sát thay đổi bên trong hành vi website để xác định xem username có hợp lệ hay không
- Liệt kê useranmeration thường xảy ra ở login page. Ví dụ, khi bạn nhập vào một usernam hợp lệ nhưng mật khẩu thì sai, hay trên form đăng ký khi bạn nhập vào trên một user đã tồn tại thì nó sẽ báo lỗi. Điều này giúp thực hiện brute-force dễ dàng hơn bởi atker có thể nhanh chóng tạo ra một danh sách ngắn tổng hợp các username hợp lệ.
- Trong khi brute-force trên một login page, bạn nên để ý các sự khác biệt bên trong:
  - Status Code (Mã trạng thái): Trong suốt một brute-force atk, HTTP status code trả về có thể giống nhau trong phần lớn các dự đoán bởi vì hầu hết chúng là sai (ví dụ là 404). Nếu status code trả về khác phần còn lại thì thì khả năng rất cao là username là đúng. Đây là. Cái này có thể khắc phục bằng cách để website trả về cùng một status code bất kể đúng hay sai
  - Error Message: Đôi khi error message trả về sẽ khác nhau phụ thuộc vào username và pass cùng sai hay chỉ pass sai. Điều này có thể khắc phục để cho hệ thống trả về cùng một message cho cả 2 trường hợp, kiểu như "tài khoản hay mật khâu sai", hay thâm chí là bỏ luôn message
  - Thời gian phản hồi: Nếu hầu hết các request được xử lý bằng cùng một response time, thời gian này mà khác tức là có gì đó khác xảy ra trong hệ thống. Ví dụ, một web có thể sẽ kiểm tra liệu pass có đúng không nếu username hợp lệ. Bước phụ này thường gây ra response time tăng một chút. Không phải lúc nào cũng chuẩn, nhưng một atker có thể khiến nó delay hơn rõ ràng nhất bằng cách một dãy pass dài ngoằng khiến website xử lý pass lâu hơn
## Lab
- Lab này tồn tại lỗ hổng để liệt kê username và pass brute-forcce atk. Nó có một account với username dễ đoán và pass, có thể tì thấy trong wordlist được cung cấp
- Để giải lab này, liệt kê các username hợp lê, brute-force pass của chúng, rồi truy cập vào account page
- Ok vì đây là lần đầu sử dụng cái Tab Intruder nên tôi đã phải đọc solution để hiểu cách làm
- Đầu tiên ta đăng nhập bằng username với pass bất kỳ :)) nếu bạn may mắn vcl thì nó có thể đúng :)) chứ gần như chắc chắn là nó sẽ báo sai
- Lúc đó, Burp sẽ ghi lại hoạt động login của ta trong Proxy - Proxy History
- Tìm ở cuối dòng giá trị của tham số username (là cái username bạn nhập vào), chuột phải rồi send to Intruder
- Khi vào Intruder, cái username ta vừa chọn có thêm giá trị $ 2 bên (thể hiện cho giá trị động). Pass ta không đổi vì mục tiêu của ta là tìm username trước
- Chuyển sang tab Payload của Intruder, set Payload Type là Simple List. Sau đó copy wordlist username được cung cấp rồi paste vào payload setting rồi start atk
- Sau khi xong, ta để ý cột Length có một (hoặc nhiều) entries dài hơn hẳn phần còn lại, so sánh response của payload này với phần còn lại, thì các response còn lại trả về ```invalid username```, còn payload này thì là ```incorrect password``` cho thấy nó là username hợp lệ
- Quay lại tab Position của Intruder. Chọn clear, rồi thay đổi ```username``` thành username hợp lệ vừa tìm được. Lần này làm như bước đầu nhưng là chọn với ```password```
