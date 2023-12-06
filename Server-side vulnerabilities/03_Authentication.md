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
## Điểm khác biệt giữa xác thực và uỷ quyền (Authentication & Authorization)
- Xác thực là một tiến trình xác minh một user là chính họ (tức là người login có phải chính chủ không). Còn uỷ quyền thì là xác minh xem user đó có quyền để thực hiện một chức năng nào đó không
- Ví dụ, xác thực xác định liệu ai đó đang cố gắng truy cập vào một web với username ```Carlos123``` có thực sự là người dùng đã tạo tài khoản này không
- Một khi ```Carlos123``` được xác thực, quyền của họ quyết định xem họ được uỷ quyền những gì. Ví dụ, họ có thể được uỷ quyền để truy cập vào thông tin cá nhân của các user khác, hay thực hiện hành đồng như xoá user's account

# Lỗ hổng xác thực pass
## Brute-force atk (lỗ hổng đăng nhập)
- Một brute-force atk là khi một atker sử dụng một hệ thống thử nghiệm và test lỗi để đoán ra các tên người dùng đang tồn tại. Kiểu tấn công này thông thường được thực hiện tự động với wordlist các username và password. Tự động hoá tiến trình này, đặc biệt là sử dụng các công cụ chuyên dụng, sẽ cho ra khả năng thử login ở tốc độ cao
- Brute-force không phải lúc nào cũng là đoàn username và pass một cách hoàn toàn random. Bằng cách sử dụng logic và những thông tin có sẵn, atker có thể tinh chỉnh brute-force để khiến việc đoán tốt hơn. Nó cải thiện dáng kể độ hiệu quả của đợt tấn công.
- Website mà chỉ dựa vào login bằng pass thông thường mà không có thêm phương thức nào khác để xác thực user có thể sẽ dễ dàng bị khai thác nếu không thực hiện các biện pháp chống brute-force

### Brute-forcing usernamé
- Username cực kì dễ đoàn nếu chúng tuân thủ một mô hình dễ nhận biết, như là một email address. Ví dụ, ta thường thấy mọi người login theo format này ```firstname.lastname@somecompany.com```. Tuy nhiên, thậm chí không có một hình mẫu rõ ràng, thỉnh thoảng tài khoản đặc quyền cao cũng dùng những từ dễ đoàn, như là ```admin``` hay ```administrator```
- Trong suốt quá trình theo dõi, kiểm tra nếu website tiết lộ username một cách công khai. Ví dụ, bạn có thể truy cập vào tài khoản mà không cần login không? Thâm chí nội dung thực tế của profile bị ẩn, tên được sử dụng trong profile đôi khi lại giống username.Bạn cũng nên kiểm tra HTTP response để xem email/username nào đang tồn tại. Thỉnh thoảng, response bao gồm email với user đặc quyền cao, như là admin hay IT support

### Brute-forcing password
- Pass có thể brute-force tương tự, với độ khó phụ thuộc vào độ mạnh của pass. Nhiều website áp dụng một số dạng chính sách pass, điều này yêu cầu user để tạo nên pass có độ phức tạp cao, ít nhất thì sẽ gây khó khăn khi chỉ dùng brute-force để tấn công. Điều này thường yêu cầu pass có những đặc điểm sau:
  - Số kí tự ít nhất
  - Kết hợp kí tự hoa và thường
  - Ít nhất một kí tự đặc biệt
- Tuy nhiên, trong khi mật khẩu phức tạp khó cho máy tính crack một mình, ta có thể sử dụng những kiến thức cơ bản về hành vi con người để khai thác lỗ hổng mà user vô tình làm lộ trong hệ thống. Thay vì tạo một pass mạnh với những kí tự ngẫu nhiên, user thường dùng pass dễ nhớ và cố gắng làm đó phù hợp với chính sách pass của hệ thống. Ví dụ, nếu ```mypassword``` không được cho phép, thì họ sẽ dùng kiểu ```Mypassword!``` hay ```Myp4$$w0rd```
- Sự hiểu biết về thông tin xác thực có khả năng và mô hình dễ đoán có nghĩa là brute-force atk có thể thươngf phức tạp hơn rất nhiều, và vì vậy rất hiệu quả, hơn là thử bừa mọi thứ có thể

### Liệt kê username
- Liệt kê username là khi một atker có thể quan sát thay đổi bên trong hành vi website để xác định xem username có hợp lệ hay không
- Liệt kê useranmeration thường xảy ra ở login page. Ví dụ, khi bạn nhập vào một usernam hợp lệ nhưng mật khẩu thì sai, hay trên form đăng ký khi bạn nhập vào trên một user đã tồn tại thì nó sẽ báo lỗi. Điều này giúp thực hiện brute-force dễ dàng hơn bởi atker có thể nhanh chóng tạo ra một danh sách ngắn tổng hợp các username hợp lệ.
- Trong khi brute-force trên một login page, bạn nên để ý các sự khác biệt bên trong:
  - Status Code (Mã trạng thái): Trong suốt một brute-force atk, HTTP status code trả về có thể giống nhau trong phần lớn các dự đoán bởi vì hầu hết chúng là sai (ví dụ là 404). Nếu status code trả về khác phần còn lại thì thì khả năng rất cao là username là đúng. Đây là. Cái này có thể khắc phục bằng cách để website trả về cùng một status code bất kể đúng hay sai
  - Error Message: Đôi khi error message trả về sẽ khác nhau phụ thuộc vào username và pass cùng sai hay chỉ pass sai. Điều này có thể khắc phục để cho hệ thống trả về cùng một message cho cả 2 trường hợp, kiểu như "tài khoản hay mật khâu sai", hay thâm chí là bỏ luôn message
  - Thời gian phản hồi: Nếu hầu hết các request được xử lý bằng cùng một response time, thời gian này mà khác tức là có gì đó khác xảy ra trong hệ thống. Ví dụ, một web có thể sẽ kiểm tra liệu pass có đúng không nếu username hợp lệ. Bước phụ này thường gây ra response time tăng một chút. Không phải lúc nào cũng chuẩn, nhưng một atker có thể khiến nó delay hơn rõ ràng nhất bằng cách một dãy pass dài ngoằng khiến website xử lý pass lâu hơn
#### Lab
- Lab này tồn tại lỗ hổng để liệt kê username và pass brute-forcce atk. Nó có một account với username dễ đoán và pass, có thể tì thấy trong wordlist được cung cấp
- Để giải lab này, liệt kê các username hợp lê, brute-force pass của chúng, rồi truy cập vào account page
- Ok vì đây là lần đầu sử dụng cái Tab Intruder nên tôi đã phải đọc solution để hiểu cách làm
- Đầu tiên ta đăng nhập bằng username với pass bất kỳ :)) nếu bạn may mắn vcl thì nó có thể đúng :)) chứ gần như chắc chắn là nó sẽ báo sai
- Lúc đó, Burp sẽ ghi lại hoạt động login của ta trong Proxy - Proxy History
- Tìm ở cuối dòng giá trị của tham số username (là cái username bạn nhập vào), chuột phải rồi send to Intruder
- Khi vào Intruder, cái username ta vừa chọn có thêm giá trị $ 2 bên (thể hiện cho giá trị động). Pass ta không đổi vì mục tiêu của ta là tìm username trước
- Chuyển sang tab Payload của Intruder, set Payload Type là Simple List. Sau đó copy wordlist username được cung cấp rồi paste vào payload setting rồi start atk
- Sau khi xong, ta để ý cột Length có một (hoặc nhiều) entries dài hơn hẳn phần còn lại, so sánh response của payload này với phần còn lại, thì các response còn lại trả về ```invalid username```, còn payload này thì là ```incorrect password``` cho thấy nó là username hợp lệ ![image](https://github.com/Myozz/Web_Applications/assets/94811005/78fae010-2f67-4789-87f1-fbe8d0553f20)
- Quay lại tab Position của Intruder. Chọn clear, rồi thay đổi ```username``` thành username hợp lệ vừa tìm được. Lần này làm như bước đầu nhưng là chọn với ```password```

### Lỗi brute-force protection
- Rất có khả năng một brute force atk sẽ liênquan đến nhiều lần thử sai trước khi thành công. Một cách logoc, brute force protection xoay quanh việc cố gắng khiến nó  tinh vi nhất có thể để tự động xử lý  và làm chậm tỉ lệ mà ở đó một atker có thể cố gắng login. Có 2 cách phổ biến nhất để ngăn chặn brute force là
  - Khoá tài khoản mà remote user đang cố gắng truy cập nếu họ fail quá nhiều lần
  - Block IP nếu login quá nhanh
- Hai cách trên khá hiệu quả, nhưng không có nghĩa là không có lỗ hổng, đặc biệt nếu flawed logic được thực hiện
- Ví dụ, bạn có thể đôi khi thấy rằng IP của bạn bị blocked nếu bạn fail quá nhiều login. Trong một số triển khai, bộ đếm số lần fail reset nếu IP owner đăng nhập thành công. Có nghĩa là một atker có thể log in vào acc của chính họ sau vài lần fail nhất định để reset bộ đếm
- Trong trường hợp, chỉ đơn thuần là thêm tài khoản của bạn vào wordlist để reset bộ đếm để khiến phương thức def này vô dụng
### Account locking
- Một cách mà ở đó các websites cố gắng ngăn chặn brute force là khoá acc nếu một số tiêu chí đáng ngờ xuất hiện. Cũng như các lỗi login thông thường, response từ server cho biết tài khoản đã bị khoá cũng có thể giúp atker liệt kê username
- Khoá tài khoản mục đích là nhằm bảo vệ khỏi những cuộc atk brute force một tài khoản cụ thể. Truy nhiên, cách tiếp cận này không thể ngăn chặn thoả đáng các cuộc atk brute force trong đó atker cố gắng chiếm quyền truy cập vào một tài khoản random
- Ví dụ, phương thức dưới đấy có thể sử dụng để xử lý loại protection này
  - Triển khai một danh sách username có thể hợp lệ.
  - Dùng một short list mà bạn cho là sẽ có ít nhất một user có nó. Điều quan trọng là, số pass bạn lựa chọn không được nhiều hơn số giới hạn số lần sai pass.
  - Sử dụng tool như Burp Intruder, thử cách pass được chọn với mỗi username. Bằng cách này, bạn có thể brute force mỗi acc mà không bị khoá ac. Bạn chỉ cần một user để sử dụng một trong ba pass để chôm được tài khoản
### User input limit
- Đại khái là thay vì block acc thì nó sẽ block IP
- Đương nhiên vẫn sẽ có cách bypass, ví dụ gửi nhiều pass chỉ trong một request, như dưới đây

![image](https://github.com/Myozz/Web_Applications/assets/94811005/af076674-d4fb-4ee2-ab7a-9e9c0312b32f)

## HTTP basic auth
- Mặc dù cũ vl rồi nhứng tính đơn giản và dễ triển khai tương đối của HTTP basic auth. Trong HTTP basic auth, máy chủ nhận một auth token từ server, được cấu trúc bằng cách nối username với pass, rồi encode bằng Base64. Token này được lưu trữ và quản lý bởi trình duyệt, mà nó có thể tự động thêm token vào bất kì header xác thực nào trong mọi request như dưới đây:

```Authorization: Basic base64(username:password)```
- Vì một vài lí do, điều này thường không được xem xết là một phương thức xác thực bảo mật. Đầu tiên, nó liên quan đến việc gửi lặp đi lặp lại user login credential với mỗi request. Nếu web cũng không triển khai HSTS (HTTP Strict Transport Security - một chính sách bảo vệ web), thông tin user có thể bị ghi lại trong một đợt atk man-in-the-middle (trung gian)
- Ngoài ra, việc triển khai HTTP basic auth thương không hỗ trợ brute-force protection. Vì token chứa những giá trị tĩnh, điều này có thể bị lợi dụng để brute force
- HTTP basic auth cũng có thể bị lợi dụng để khai thác các lỗ hổng liên quan đến session
- Trong một vài trường hợp, khai thác HTTP basic auth chỉ có thể cấp quyền cho atker truy cập vào cái page cùi bắp. Tuy nhiên, ngoài việc cung cấp một môi trường để atk, thông tin xác thực được sử dụng theo cách này có thể bị tái sử dụng ở page khác (ví dụ hack được user:pass ở page cùi bắp, nhưng thằng user set ở page nào cũng vân pass đấy thì bú)

# Lỗ hổng xác thực đa bước
- Ở phần này, ta sẽ xem một vài lỗ hổng có thể xảy ra trong xác thực đa bước.
- Nhiều web dựa trên xác thực một bước (là nhập pass). Tuy nhiên, một số khác bắt buộc user phải xác thực xử dụng nhiều bước
- Xác thực sinh trắc học là không thực tế với đa số web. Tuy nhiên, dễ thấy răng cả xác thực bắt bược và 2FA phụ đều dựa trên **nhưng gì bạn biết** và **những gì bạn có**. Điều này thương bắt buộc user nhập cả pass và một mã xác thực tạm thời từ một thiết bị vật lý họ sở hữu
- Lưu ý răng xác thực đa bước chỉ phát huy tác dụng khi xác minh trên nhiều yết tố khác nhau. Xác minh cùng một yếu tố bằng 2 cách khác nhau. Xác thực cùng một yếu tố theo 2 cách khác nhau không phải 2FA đúng nghĩa. 2FA dựa trên email là một ví dụ cụ thể. Mặc dù user phải cung cấp một pass và một mã xác thực, nhưng việc truy cập lấy mã xác thực cũng là bằng cách xác thực đăng nhập email. Vì vậy, đây chỉ đơn giản là xác thực một yếu tố 2 lần

# 2FA auth token
- Mã xác thực thương được đọc bởi user từ một thiết bị vật lú
- Có một số website gửi mã xác thực qua SMS. Đúng là nó vẫn xác minh dựa trên yếu tố **Những gì bạn có**, nhưng nó có thể bị lợi dụng. Đầu tiên, code được gửi qua SMS (SĐT) chứ không phải bản thân thiết bị. Điều này tạo ra khả năng để mã xác thực bị chặn lại. Cũng có rủi ro khi thay SIM, atker có thể chôm SIM lấy sđt nhận code :|| nghe nguy hiểm quá
## Bypass two-factor authentication (Bypass xác minh 2 bước)
- Đôi khi, việc triển khai 2FA sẽ có những sai sót khiến nó có thể bị bypass dễ dàng.
- Nếu user cần nhập pass trước khi phải nhập mã xác minh 2fA ở 2 trang riêng biệt, user thực ra đã ở trạng thái "đã login" khi nhập mật khẩu xong (nhập 2FA chỉ đơn giản là để hiện nó ra). Trong trường hợp này, có thể ta sẽ trực tiếp skip 2FA ngay sau khi nhập mật khẩu. Thỉnh thoảng, bạn sẽ nhận ra một website không thực sự kiếm tra xem bạn có hoàn thành bước 2 hay không trước khi loading page
### Lab
- Việc ta cần làm là bypass 2FA của lab này, và đột nhập vào tài khaonr carlos
- Login vào account wiener của mình (thực ra không log cũng được, chủ yếu là để lấy cái đường path vào account page thôi)
- Login vào tài khoản carlos, nó sẽ bắt điền 2FA nhưng cực kì ngu, ta có 2 hướng giải quyết đoạn này
  - Dùng path ta xem được bên wiener, là ```/my-account```
  - Có cái nút Back to Home, dùng nút đấy để back về trang chủ rồi click lại vào My Account là xong ![image](https://github.com/Myozz/Web_Applications/assets/94811005/7436221c-1707-455c-97ef-4878f9351b42)
## Lỗi logic trong 2FA
- Đôi khi lỗi logic trong 2FA có nghĩa là sau khi một user hoàn thành bước login, website sẽ không thực sự kiểm tra bước thứ 2
- Ví dụ, user login với thông tin xác thực thông thường ở bước đầu tiên như dưới đây

        POST /login-steps/first HTTP/1.1
        Host: vulnerable-website.com
        ...
        username=carlos&password=qwerty
- Chúng sau đó được chỉ định cho một cookie mà có liên quan tới acc của chúng, trước khi được đem tới bước 2 (tức là lúc này nó vốn dĩ đã login vào rồi)

      HTTP/1.1 200 OK
      Set-Cookie: account=carlos
      
      GET /login-steps/second HTTP/1.1
      Cookie: account=carlos
- Khi nhập code, request sử dụng cookie để xác định xem user đang cố gắng truy cập vào acc nào :))

      POST /login-steps/second HTTP/1.1
      Host: vulnerable-website.com
      Cookie: account=carlos
      ...
      verification-code=123456
- Trong trường hợp này, một atker có thể login sử dụng thông tin của họ nhưng rồi thay đổi giá trị của cookie thành các username khác khi submit

      POST /login-steps/second HTTP/1.1
      Host: vulnerable-website.com
      Cookie: account=victim-user
      ...
      verification-code=123456
- Điều này cực văn lê minh kỳ nguy hiểm nếu atker sau đó có thể brute force mã xác thực, và :|| ye
## Brute-force 2FA code
- Làm tương tự như với pass, web cần phải có các cơ chế để ngăn chặn brute force 2FA. Điều này đặc biệt quan trọng bởi vì code đường là 4-6 chữ số. Nếu k có biện pháp bảo vệ thì bypass ez
- Một số web nỗ lực ngăn cản điều này bằng cách tự động log out một user nếu họ nhập sai một số lần nhất định. Đây là cách không hiệu quả trên thực tế bởi một atker có thể gửi cùng lúc nhiều request (Burp Intruder)

# Lỗ hổng trong các cơ chế xác thực khác
## Giữ người dùng đăng nhập
- Một cái tính năng khá phổ biến là khi đóng trình duyệt, mà khi vào lại ta không cần phải login lại
- Tính năng này thường thực hiện bằng cách tạo một "remember me" token, cái mà được lưu trữ trong một cookie. Nếu lấy được cookie này thì ta có thể bypass được bước login, trên thực tế thì cookie không thể đoán được. Tuy nhiên, một số web tạo cookie dựa trên một sự kết nối có thể đoán được của các giá trị tĩnh, như là username và một mốc thời gian. Một số thậm chí sử dụng pass làm một phần của cookie. Sẽ là mối nguy hại nếu một atker có thể tạo tài khoản của riêng rồi dùng cookie của họ để tìm ra quy luật của cookie. Rồi từ đó brute force các tài khoản khác
- Một số web giả sử rằng nếu cookie được mã hoá bằng cách nào đó nó sẽ không thể đoán kể cả khi nó sử dụng dữ liệu tĩnh. Điều này nghe có vẻ đúng nếu được thực hiện đúng cách, thường thì việc mã hoá cookie sử dụng một dạng mã hoá two-way đơn giản giống base64 không cung cấp bất cứ sự bảo mật nào. Thậm chí sử dụng một loại mã hoá hash một chiều thì vẫn chưa hoàn toàn an toàn. Nếu atker có thể xác định được thuật toán hash, và không có salt được sử dụng, họ có thể brute force cookie bằng cách hash wordlist của họ. Phương thức này có thể sử dụng để bypass giới hạn lần login nếu một giới hạn tương tự không được áp dụng với cookie
- Thậm chí nếu atker không thể tạo tài khoản của riêng họ, họ có lẽ vẫn có thể khai thác lỗ hổng này. Sử dụng một kĩ thuật phổ biến như XSS, một atker có thể lấy cắp được cookie lưu trữ pass của người dùng khác và suy ra cấu trúc của cookie từ đó. Nếu website được xây dựng sử dụng open-source framework, những chi tiết quan trọng trong cấu trúc của cookie thậm chí còn được public
- Trong một vài trường hợp hiếm gặp, nó có lẽ sẽ khả thi để lấy được luôn pass của user từ cookie, thậm chí nếu nó được hash đi nữa. Những phiên bản hash của của các pass phổ biến là có sẵn, vậy nên nếu một user pass xuất hiện trong list này, việc decrypt hash có thể đơn giản như là paste chúng vào công cụ tìm kiếm. Điều này chứng tỏ sự quan trọng của salt trong mã hoá

## Reset user pass
- Trong thực tế, có một lựa chọn là quên pass, vậy tức là có một cách để reset pass. Thông thường phương thức xác thực bằng pass là không thể rồi, web phải có một phương thức thay thế để đảm bảo rằng đấy chính là chủ nhân tìa khoản. Vì lí do này, tính năng reset pass vốn rất nguy hiểm và cần được cải thiện về bảo mật
### Gửi pass bằng email
- Thường thì có một số web sẽ tạo ra một pass mới và gửi cho user qua mail
- Nói chung việc gửi pass kiểu này thường bị tránh :||. Trong trường hợp này, pass được gửi sẽ hết hạn trong khoảng time ngắn hay user phải change pass ngay lập tức. Mặt khác, điều này thì lại ngon nghẻ để chơi man-in-middle atk
- Email thường không được xem xét được coi là an toàn vì vì inbox vừa ổn định vừa không được thiết kế để lưu trữ an toàn thông tin bí mật. Nhiều user cũng được đồng bộ tự động với inbox của họ giữa nhiều thiết bị
### Reset pass qua URL
- Một phương thức reset pass khác là gửi một URL đặc biệt tới user để dẫn đến trang reset pass. Nếu URL này mà dễ đoán thì rõ ràng rất nguy hiểm, kiểu như:

      http://vulnerable-website.com/reset-password?user=victim-user
- Trong ví dụ này, một atker có thể thay đổi tham số ```user``` để bất kì username nào xác định được. Chúng có thể sẽ đưa thẳng tới trang reset pass của user đấy
- Túm lại nếu muốn dùng cách reset này thì phải đảm bảo về độ phức tạp, khó đoán của URL. Trong th tốt nhất, URL này nên không cung cấp bất cứ gợi ý nào về user

      http://vulnerable-website.com/reset-password?token=a0ba0d1cb3b63d13822572fcff1a241895d893f659164d4cc550b421ebdd48a8
- Khi user tuy cập URL, hệ thông nên kiểm tra nếu token này tồn tại trong back-end và, nếu vậy; pass của user nào sẽ chuẩn bị được reset
- Tuy nhiên, một số web cũng không thể xác thức lại token ghi form được gửi đi. Trong trường hợp này, một atker có thể vào reset form từ chính account của họ, xoá token và tận dụng page này để reset một user's pass bất kỳ
- Nếu URL trong reset emaiol được tạo ra động, nó có thể gây ra lỗ hổng để thực hiện pass reset poisoning. Trong tình hướng này, một atkẻ có thể chôm token của người khác và sử dụng để change pass
## Changing user pass
- Tiêu biểu, đổi pass liên quan đến việc nhập pass hiện tại và nhập 2 lần pass mới. Những page này cũng dùng cùng cái phương thức chess pass như login form. Vì vậy, những page này có thể có lỗ hổng giống nhau
- Chức năng thay pass có thể nguy hiểm nếu nó cho phép một atker truy cập trực tập mà không cần log in. Ví dụ, nếu một username được cung capá trong một field ẩn, một atker có thể sửa giá trị request thành mục tiêu bất kì. Điều này có thể được thực thi để liệt kê và brute force pass
