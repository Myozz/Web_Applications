# Race conditions (Điều kiện thực hiện)
- Là một loại lỗ hổng phổ biến liên quan đến lỗi logic kinh doanh. Chúng xảy ra khi web thực hiện request kiểm nhiểm mà không có những biện pháp bảo mật đầy đủ. Điều này có thể dẫn tới các mối đe doạ tương tác với cùng một dữ liệu tại cùng một thời điểm, hậu quả là sự "va chạm" (collision) mà có thể gây ra những hành vi ngoài ý muốn và có thể bị khai thác cho mục đích xấu
- Khoảng thời gian xảy ra va chạm còn được gọi là **race window** (cửa sổ đua). Khoảng này có thể là một phần bao nhiêu đó của một giây giữa 2 tương tác với dtb.
- **Ví dụ**: Như nhũng lỗi logic khác, tác đọng của một race condition phụ thuộc nhiều vào app và tính năng nhất định mà tại đó nó xảy ra
- Ở phần này, ta sẽ nghiên cứu làm sao để xác định và khai thác các loại khác nhau của race condition. Ta sẽ xem cách mà Burp Tool hỗ trợ ta vượt qua thử thách để thực hiện atk cơ bản, cộng thêm một phương pháp đã được kiểm/thử nghiệm mà cho phép bạn phát hiện được tình trạng của race condition bên trong các tiến nhưng đa bước ẩn (hidden multi-step). Những thứ này vượt qua giới hạn mà bạn có thể quen thuộc?

# Limit overrun race condition (Điều kiện thực hiện vượt qua giới hạn)
- Loại race condition được biết tới nhiều nhất cho phép bạn vượt qua một số loại giới hạn được sử dụng bởi bussiness logic của app
- Ví dụ, hãy xét một cửa hàng online cho phép bạn nhập một promo code để có được phiếu giảm giá. Để áp dụng phiếu giảm giá đó, ap sẽ thực hiện những tác vụ bậc cao:
  - Kiểm tra xem bạn đã sử dụng code chưa
  - App dụng giảm giá vào sản phẩm
  - Cấp nhật các bản ghi trong dtb để cho biết rằng ta đã sử dụng code này rồi
- Nếu bạn muốn sử dụng lại code này, sẽ có một kiểm tra ở đầu tiến trình nhằm ngăn chặn bạn làm diều đó ![image](https://github.com/Myozz/Web_Applications/assets/94811005/afb28664-7fe8-41d9-8218-e3ad364c110c)
- Giờ hãy xét tới điều gì sẽ xảy ra nếu một user mà chưa từng sử dụng code trước đó nhập cùng một code 2 lần gần như cùng một lúc ![image](https://github.com/Myozz/Web_Applications/assets/94811005/b49195b5-0654-4a55-b4e8-b75cd98becd5)
- Như bạn có thể thấy, app chuyển đổi thông qua trạng phụ tạm thời. Trạng thái mà đi vào rồi exit trước khi request đang chạy hoàn thành. Trong trường hợp này, trạng thái phụ bắt đầu khi server bắt đầu thực hiện request đầu tiền, và kết thúc khi nó cập nhật dữ liệu trong dtb để xác định xem ta sử dụng code hay chưa. Điều này cho ta một cửa sổ đua (race window) nhỏ trong suốt thời điểm này mà bạn có thể dùng code bao nhiêu lần tuỳ ý
- Có nhiều biến thể của kiểu tấn công này, boa gồm:
  - Dùng gift card nhiều lần
  - Đánh giá sản phẩm nhiều lần
  - Rút tiền, chuyển tiền vượt quá số dư
  - Tái sử dụng giải CAPTCHA
  - Bypass giới hạn của anti-brute-force
# Phát hiện và khai thác limit overrun race condition với Burp Repeater
- Tiến trình phát hiện và khai thác limit overrun race condition tương đối đơn giản. Ở điều khiến cấp cao, tất cả những gì chúng ta cần làm là:
  - Xác định một endpoint sử dụng đơn (single-use) hay giới hạn tỉ lệ (rate-limited) mà có một vài loại tác động bảo mật hay ua7 mục đích hữu ích khác
  - Multiple request tới endpoint này với tốc độ ánh sáng và xem vượt quá giối hạn4
- Thử thách cơ bản là timing request nên cần có ít nhất cần có 2 race window phối hợp, gây ra một đợt va chạm. Cửa sổ này thưởng chỉ vài mili giấy và thầm chí có thể ngắn hơn nữa
- Thậm chí nếu gửi tất cả request cùng lúc, trong thực tế có nhiều yếu tố không thể kiểm soát hay đonas trước ảnh hướng tới thời điểm server thực hiện mỗi request và theo thứ tự như nào ![image](https://github.com/Myozz/Web_Applications/assets/94811005/1b26081a-2dc6-41cc-b837-1d8549687f88)
- Burp có một công cụ mạnh mẽ là **Burp Repeater** cho phép bạn dễ dàng gửi một loạt request theo một cách mà giảm thiếu tối đa tác động của các yếu tố trên (network jilter). Burp tự động điều chỉnh kĩ thuật mà nó sử dụng để phù hợp với HTTP version được hỗ trợ
  - Với HTTP/1, nó sử dụng kĩ thuật đồng bộ byte cuối đơn giản (classic last-byte synchronization technique)
  - Với HTTP/2, nó sử dụng kĩ thuật tấn cống gói tin đơn (single-packet atk technique), được phát hiện bởi PortSwigger Research tại Black Hat USA 2023 (ngon)
  - Single-packet atk cho phép bạn hoàn toàn vô hiệu hoá nhiều từ network jilter bằng cách sử dụng TCP packet đơn để hoàn thành 20-30 request cùng lúc ![image](https://github.com/Myozz/Web_Applications/assets/94811005/10bf0567-895e-4856-a33b-4880c91f323e)
- Mặc dù bạn có thể sử dụng 2 request để kích hoạt một exploit, gửi một lượng lớn request như vậy giúp giảm thiểu trễ nội bộ (internal lantency), hay còn gọi là server-side jilter. Điều này đặc biệt hữu dụng trong suốt quá trình khám văn phá.
> Chi tiết về cách sử dụng tính năng mới của Burp Repeater để gửi nhiều request cùng lúc, xem ở [dấy](https://portswigger.net/burp/documentation/desktop/tools/repeater/send-group#sending-requests-in-parallel)
> Để có một cài nhìn rõ ràng hơn về single-packet atk, và chi tiết hơn về phương pháp, hay xem ở [đây](https://portswigger.net/research/smashing-the-state-machine). 
## Lab
- Luồng mua hàng của lab này chứa race condition cho phép bạn mua hàng phá giá
- Để hoàn thành lab, hãy mua thành công một **Lightweight L33t Leather Jacket**
- Bạn có thể login vào ```wiener:peter```
- Ok trước vào là ta login vào đã
- Khi login thành công thì ta thấy có 2 thông tin có vẻ là sẽ có ích trong quá trình thực hiện race condition là mã giảm giá và số dư tài khoản ![image](https://github.com/Myozz/Web_Applications/assets/94811005/71545433-8246-47a7-bd05-d3e64c4b3359)
- Thử quay lại home và kiểm tra mặt hàng cần mua nào
- Thêm vào giỏ hàng mà ta biết được giá của sản phẩm là 1337$, gấp khoảng 30 lần số dư của ta ![image](https://github.com/Myozz/Web_Applications/assets/94811005/ba58b59c-fa04-4f26-8100-813bf4b0364c)
- Mã giảm giá giúp ta giảm 20% tức là nếu ta tìm được cách apply nó tầm hơn chục lần thì có thể mua mà không cần 1337$
- Ta hãy thử intercept cái nút Apply xem
- Tôi đã chuyển sang Repeater, chuột phải vào tab rồi ```add to group```, và ```CTRL+R``` đến khi ra tầm vài chục tab :)) ![image](https://github.com/Myozz/Web_Applications/assets/94811005/6d5195c7-ff28-4f37-aa80-55a82d6f9dce)
- Ở nút send lúc này sẽ có các tuỳ chọn, hãy xem ở [dấy](https://portswigger.net/burp/documentation/desktop/tools/repeater/send-group#sending-requests-in-parallel) để biết nên chọn cái này nhé, rồi nhấn send thôi
- **NOTE**: Nếu gửi nhiều quá thì có thể nó sẽ bị sida đấy :)) gửi vừa đủ thôi nhé, tôi gửi 25 request là thấy đủ rồi

# Phát hiện và khai thác limit overrun race condition với Turbo Intruder
- Ngoài Burp Repeater, nhà phát triển cũng có thêm một extension là ```Turbo Ỉntruder``` để hỗ trợ kĩ thuật này
- Turbo Intruder yêu cầu một số khả năng của Python, nhưng phù hợp với những atk phức tạp hơn, như những cái mà cần phải gửi lại nhiều lần, timing request so le nhau, hay gửi một lượng lớn request
- Để sử dụng single-packet atk trong Turbo Intruder
  - Đảm bảo răng mục tiêu hỗ trợ HTTP/2. single-packet atk không phù hợp với HTTP/1
  - Set ```engine=Engine.BURP2``` và ```concurrentConnections=1``` config option cho request engine
  - Khi đợi request, nhóm chúng bằng cách phân chúng tới một cổng được đặt tên sử dụng đối số ```gate``` cho phương thức ```engine.queue()```
  - Để gửi tất cả request trong group, mở cổng tương ứng với phương thức ```engine.openGate()```

        def queueRequests(target, wordlists):
        engine = RequestEngine(endpoint=target.endpoint,
                                concurrentConnections=1,
                                engine=Engine.BURP2
                                )
        
        # queue 20 requests in gate '1'
        for i in range(20):
            engine.queue(target.req, gate='1')
        
        # send all requests in gate '1' in parallel
        engine.openGate('1')
- Để thêm thông tin, hãy xem ```race-single-packet-attack.py``` bên trong thư mục ví dụ mặc định của Turbo Intruder
## Lab
- Lab này có cơ chế chống brute-force. Tuỳ nhiên thì có thể bypass bằng cách dùng race condition
- Để giải lab
  - Tìm ra cách khai thác race condition để bypass rate limit
  - Brute-force password của ```carlos```
  - Login và truy cập vào admin pane;
  - Xoá carlos
- Có thể login vào ```wiener:peter```
- Intercept nút login trong trang đăng nhập, chuột phải - extensions - turbo intruder - send to...
- Sửa tham số password thành %s ![image](https://github.com/Myozz/Web_Applications/assets/94811005/579d0140-7c73-455d-9375-f1ae11961c44)
- Dùng đoạn code

      def queueRequests(target, wordlists):

        # as the target supports HTTP/2, use engine=Engine.BURP2 and concurrentConnections=1 for a single-packet attack
        engine = RequestEngine(endpoint=target.endpoint,
                               concurrentConnections=1,
                               engine=Engine.BURP2
                               )
        
        # assign the list of candidate passwords from your clipboard
        passwords = wordlists.clipboard
        
        # queue a login request using each password from the wordlist
        # the 'gate' argument withholds the final part of each request until engine.openGate() is invoked
        for password in passwords:
            engine.queue(target.req, password, gate='1')
        
        # once every request has been queued
        # invoke engine.openGate() to send all requests in the given gate simultaneously
        engine.openGate('1')
    
    
      def handleResponse(req, interesting):
          table.add(req)
- Copy wordlist vào clipboard rồi attack, rồi sẽ ra thôi

# Hidden multi-step sequences (Trình tự đa bước ẩn)
- Trên thực tế, một request đơn có thể khởi đầu một trình tự đa bước hoàn chỉnh, chuyển tiếp app thông qua nhiều trạng thái ẩn mà enter rồi lại exit trước khi request hoàn thành. Ta sẽ gọi chúng là các "trạng thái phụ"
- Bạn có thể xác định một hay nhiều HTTP request mà gây ra một tương tác với cùng một data, bạn có thể lợi dụng các trạng thái phụ này để làm lộ ra các biến thời gian nhạy cảm của loại lỗi logic phổ biến trong  quy trình làm việc đa bước. Điều này cho phép race condition khai thác vượt vượt qua limit overruns
- Ví dụ, bạn có thể quan thuộc với lỗi xác thực đa bước (MFA) mà cho phép thực hiện phần đầu tiên của login sử dụng những thông tin xác thực đã biết, sau đó điều hướng thẳng tới app thông qua forced browsing, bypass MFA một cách hiêu quả
- Đoạn pseudo-code (mã giả) dưới đấy cho ta thấy cách một web có thể dính vuln với một biến race của đợt atk này

      session['userid'] = user.userid
      if user.mfa_enabled:
        session['enforce_mfa'] = True
        # generate and send MFA code to user
        # redirect browser to MFA code entry form
- Như bạn có thể thấy, đấy là một trình tự đa bước thực tế bên trong một vòng request đơn. Quan trọng nhất, nó chuyển tiếp thông qua một trạng thái phụ mà ở đó user có một logged-in session hợp lệ, nhưng MFA là không bắt buộc. Một atker có thể khai thác điều này bằng cách gửi một login request thông qua một request tới một endpoint nhạy cảm hay đã được xác thực
- Ta sẽ nhìn vào một số ví dụ về trình tự đa bước ẩn sau, và bạn sẽ có thể liên tập khai thác chúng trong lab. Tuy nhiên, nhưng các vuln khá cụ thế, điều quan trọng đầu tiên là cần hiểu phương pháp rộng bạn, bạn sẽ cần áp dụng để xác định chúng tốt hươn, cả lab vì thực tế

# Methodology (phương pháp)
- Để phát hiện và khai thác trình tự đa bước ẩn, ta có các phương pháp sau đây
## Predict potenial collisions (Dự đoán tiềm năng va chạm)
- Kiểm tra mọi endpoint là không thực tế. Sau khi biết được site mục tiêu như thường lệ, bạn có thể giảm số endpoints mà bạn cần để test bằng cách tự hỏi bản thân nhưng câu hỏi
  - endpoint này có thực sự quan trọng không: Nếu không thực sự cần thiết thì tốt nhất nên skip
  - Có tiềm năng va chạm không?: Để va chạm thành thông, bạn cơ bản cần 2 hay nhiều request được kích hoạt trên cùng bản ghi. Ví dụ, xét biến thể triển khai đặt lại pass dưới đây ![image](https://github.com/Myozz/Web_Applications/assets/94811005/c3c5192d-dd5f-4fc6-afa9-48d74d1e3c4f)
- Với ví dự này, request nhiều đặt lại pass của 2 users khác nhau. Tuy nhiên, cái th thứ 2 cho phép bạn chỉnh sửa request của 2 users khác nhau trên cùng bản ghi
## Probe for clues (Truy tìm manh mối)
- Để có được mạnh mối, đầu tiên bạn cần đánh giá cách mà endpoint hoạt động dưới điều kiện bình thường. Bạn có thể làm điều này trong Burp Repeater bằng cách nhóm tất cả request và sử dụng ```Send group in sequence (single connnection)``` (Request theo trình tự).
- Tiếp theo, gửi một group request cùng lúc sử dụng single-packet atk (hay last-byte sync nếu là HTTP/1) để giảm thiểu network jilter. Bạn có thể làm điều này trong Repeater bằng cách chọn ```Send group in parallel``` (gửi song song). Hay có thể dùng Turbo Intruder
- Mọi thứ đều có thể là manh mối. Chỉ cần tìm kiếm một vài mẫu lệch từ những gì bạn thấy được trong suốt quá trình benchmark. Nó bao gồ một sự thay đổi trong một ay nhiều reponse, nhưng đừng quên hiệu ứng thứ 2 như nội dung email hay những thay đổi có thể tháy được trong hành vi của ứng dụng sau đó
## Prove the concept (tìm hiểu khái niệm)
- Cố gắng hiểu điều gì đang xảy ra, loại bỏ request thừa, và đảm bảo bạn có thể nhân rộng hiệu ứng
- Race condition phức tạp có thể gây ra những sự bất thường, vì vậy con đường impact mạnh nhất không phải lúc nào cũng hiện ngay trước mắt. Nó có thể giúp ta coi rằng một race conditon như một điểm yếu trong cấu trúc hươn là một lỗ hổng

# Multi-endpoint race conditions
- Có lẽ form trực quan nhất của những race condition này là những điều kiện liên quan đến việc gửi yêu cầu đến nhiều endpoint cùng lúc
- Xét về lỗi logic classic trong online stores nơi mà bạn thêm một item vào giỏ hàng, rồi trả tiền, sau đó thêm nhiều item vào giỏ trước khi force-browsing tói trang xác nhận order
- Một biến thể của vuln này có thể xảy ra khi xác nhận thanh toán và order được thực hiện trong suốt quá trình xử lý một request dơn. Trạng thái của máy cho trạng thái order có lẽ sẽ trông như này ![image](https://github.com/Myozz/Web_Applications/assets/94811005/fb438e0c-178e-436c-bc9f-06dcc53575fc)
- Trong trường hợp này, bạn có thể thêm nhiều item ơn vào giỏ trong suốt race window giữa lúc xác nhận thành toán và khi order confirm
## Căn chỉnh multi-endpoint race windows
- Khi test multi-endpoint race condition, bạn có thể bắt gặp vài vấn đề đang cố gắng để line up race windows cho mỗi request, thậm chí nếu bạn gửi nó chính xác cùng lúc sử dụng single-packet technique ![image](https://github.com/Myozz/Web_Applications/assets/94811005/4ce41a7c-824c-4306-98aa-5566c735a908)
- Vấn đề chung này trước hết được gây ra bởi 2 yếu tôs
  - **Độ trễ do kiến trúc mạng**: Ví dụ, có thể có tí delay bất kể khi nào front-end server triển khai một kết nối mới tới back-end. Giao thức được sử dụng có thể có một tác động lớn.
  - **Độ trễ do xử lý riếng endpoint**: Các endpoint khác nhau vốn đã khác nhau về time xử lý, phụ thuộc vào những hoạt động mà chúng kích hoạt
- May mắn thay, có những hướng giải quyết tiềm năng cho cả hai vấn đề trên
### Connnection warming
- Back-end connection delay không thường can thiệp với race condition atk bởi chúng thường delay các request song song như nhau, nên request luôn ở trạng thái đồng bộ
- Khá cần thiết để có thể phân biệt những delay này từ những cái được gây ra bởi yếu tố endpoint dành riêng. Một cách để làm điieuf này là "warming' kết nối với một hay nhiều request không đáng kể để thấy được nếu nó làm trơn tru phần xử lý còn lại. Trong Repeater, bạn có thể truy thêm một ```GET``` request cho homepage tới phần đầu của tab group, sau đó sử dụng ```send group in sequence (single connection)```
- Nếu request đầu tiên vẫn có một thời gian xử lý dài hơn, nhưng các request còn lại lại được xử lý cùng lúc, bạn có thể bỏ qua delay này và tiếp tục test như bình thường
## Lab liếc
- Lỗi thanh toán của lab này chứa một race condition cho phép bạn thành toán với giá cả không tưởng
- Để giải lab này. hãy thành toán quả Jacket
- 
