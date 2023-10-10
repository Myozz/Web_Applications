Toi bi buoi
# File Upload Vuln (Lỗ hổng upload file) là cl gì?
- File Up Vuln là khi một web server cho phép user upload file tới filesys mà không cần xác nhận các thông tin như name, type, content và size. Tận dụng lỗ hổng củ lìn này cho phép upload một cái img chứa đủ thể loại mã độc (may be :))). Điều này có thể bao gồm server-side script file cho phép thực thi remote code :|| đ phải tôi dịch ngu đâu mà nó khó miêu tả vcl :||
- Trong một số trường hợp, hành động upload file đã có thiệt hại rồi. Về sau làm thêm đợt atk khác  bế thêm tí HTTP request cùng file, để kích hoạt đồng được gửi lần đầu?

# File Up Vuln phát sinh như nào?
- Một website mà không có hạn chế gì thì chắc nó là thần cmn mất. Nên là thường thì dev sẽ thực hiện những gì mà họ cho là đảm bảo nhất, ít khả năng bị bypass nhất
- Ví dụ, họ có thể thêm các file type vào blacklist, nhưng lại không kiểm tra được khác biệt về cú pháp của file extension. Bất kể blacklist nào cũng đều có thể bỏ sót vài file type ẩn có thể dùng để atk :||
- Trong vài trường hợp khác, web có thể kiểm tra file type bằng cách xác thực properties mà có thể bị thao túng bởi atker dùng Burp Proxy hay Repeater
- Đặc biệt, thậm chí các biện pháp xác nhận mạnh vẫn có thể áp dụng một cách đ nhất quán qua network của host và dir mà cấu tạo nên website, hậu quả là bị khai thác, thế thôi

# Khai file upload không hạn chế để triển khai web shell
- Về góc độ bảo mật, viển cảnh tệ nhất có thể xảy ra là khi một web cho phép bạn upload server-side script, như PHP, Jav, hay Py file, và chúng đều có thể được config để thực thi như là code. Điều này khiến việc tạo web shell của riêng bạn trên server ez hơn bao giờ hết :||
- **NOTE**: Web shell là một mã đọc cho phép atker thực thi lệch trên web server đơn giản bằng cách gửi HTTP request tới đúng endpoint (điểm cuối)
- Nếu thành công upload một webshell, bạn sẽ chiếm được toàn quyền của server. Có nghĩa là bạn đọc/ghi cái b gì trên server cũng được, chôm tí dữ liệu nhạy cảm, thậm chí có thể từ đấy tấn công các server khác hay thẳng vào hạ tầng nội bộ của nạn nhân :|| và công ty họ phá sản, nạn nhân thất cmn nghiệp :||. Ví dụ, quả PHP dưới đây có thể sử dụng để độc bất kì file nào từ filesys của server

      <?php echo file_get_contents('<path của file cần đọc>'); ?>

- Một khi được upload, sẽ gửi một request cho quả file độc này, rồi trả về nội dung file cần đọc :O
- Các web shell khác cũng có cấu trúc đại khai thế

      <?php echo system($_GET['<cmd>']); ?>

- Quả shell vừa rồi dùng để thực thi một con lệnh bất kỳ bằng cách cho phép vượt qua sys cmd thông qua một tham số truy vấn như dưới đây:

      GET /example/exploit.php?command=id HTTP/1.1

## Lab đần
- Lab đần này chứa một lỗ hổng img upload. Nó không thèm kiểm tra file được upload trước khi lưu trữ chúng vào filesys (thế là đần đúng rồi)
- Để giải lab này, upload một php web shell cơ bản và sử dụng để đọc nội dung file ```/home/carlos/secret```.
- Ham làm quá quên cmnr nhưng cũng đơn giản thoi nên chả ghi lại cũng được :||

# Khai thác độ đần đét trong việc xác thực file upload
- Trong thế giới động vật :||, tìm cả giời đất chưa chắc đã thấy một website mà không có bất cứ biện pháp bảo vệ các cuộc tấn công file upload như con lab vừa rồi. Nhưng kể cả vậy cũng không có nghĩa là mình phải chịu thua chúng nó. Bạn vẫn có thể khai thác được một vài thiếu sót trong kỹ thuật để triển khai web shell

# Xác thực file type đần đét (Flawed file type validation)
- Khi gửi HTML form, trình duyệt thường gửi dữ liệu được cung cấp trong một ```POST``` request với loại nội dung (content type) ```application/x-www-form-url-encođe```. Điều này ok để gửi những đoạn text đơn giản như tên hay địa chỉ. Tuy nhiện, nó không phù hợp để gửi một lượng lớn dữ liệu nhị phân (bin data), như một file img hay tài liệu PDF. Trong trường hợp này, loại nội dung (content type) ```multipart/form-data``` được ưu tiên sử dụng

# Xác thực file type đần đét (Flawed file type validation) - tiếp
- Một form gồm các fields để upload img, cung cấp desc của ít, và nhập username. Khi gửi chúng dưới dạng form thì request sẽ trông như này ![image](https://github.com/Myozz/Web_Applications/assets/94811005/d2655a40-a7e1-403a-bc74-7e9638ddd967)
- Như đã thấy, message body được chia thành các phần riêng biệt cho một input (img, desc, username). Một phần đều có một header là ```Content-Disposition``` (Bố trí nội dung), cung mcấp một số thông tin cơ bản về input field của mỗi part. Những part riêng biệt này cũng có thể gồm ```Content-type``` của riêng chúng (như ví dụ trên thì nó cùng một content type cho cả request, trong img có một cái riêng nữa :||), sẽ thông báo cho server kiểu dữ liệu được gửi của mỗi input
- Một cách mà website có thể thử để xác thực file upload là kiểm tra ```Content-Type``` của input có đúng là type được cho phép không. Nếu server chỉ đơn giản là cho phép image file, ví dụ, nó có thể chỉ cho phép type kiểu như ```image/jpeg``` hay ```image/jpg```. Vấn đề có thể phát sinh khi giá trị của header được trust bởi server. Nếu không có tí xác thực được thực hiện để kiểm tra xem liệu file có thực sự trùng với MIME type không, kiểu def óc lìn này có thể dễ dàng bypass bằng Burp Repeater
  - Bổ túc MIME type (Multipurpose Internet Mail Extensions) - Tiện tích mở rộng mail đa năng?, là tiêu chuẩn để phân loại file type được sử dụng trên Internet
## Lab 
- Con lab này chứa lỗ hổng upload img. Nó có biên pháp ngăn chặn user upload những file khả nghi, nhưng lại phụ thuộc vào input để xác nhân điều này
- Để giải lab này, upload một php webshell đơn giản và sử dụng nó để lấy nội dung của ```/home/carlos/secret```.
- Ok triển thôi
  - Trước mắt login vào acc ```wiener``` cái đã
  - Trong account pane,, tạo một file php với nội dung ```<?php file_get_content('/home/carlos/secret'); ?>``` rồi thử upload nó lên, đương nhiên là sẽ lỗi rồi :)) bạn nghĩ nó đơn giản vậy sao
  - Vào Proxy - HTTP History để check GET/POST vừa được gửi đi, ta thấy một GET có url là ```/files/avatars/hack.php``` và một POST có url là ```my-account/avatar``` (hình như chỉ có cái thứ 2 thôi nhưng do tôi làm cái này lúc mọi thứ đã hoàn thành nên có thêm :|| ![image](https://github.com/Myozz/Web_Applications/assets/94811005/b01e26d0-b827-4436-b02a-0ef3f4c78ccf)
  - Send cái ```my-account/avatar``` sang repeater để test :|| send và check response ![image](https://github.com/Myozz/Web_Applications/assets/94811005/a9c1917c-3354-42dd-9580-552659ee332c)
    - Để ý response nói là chỉ được up file có type là ```image/jpeg``` hay ```image/png```
    - Vậy là chỉ cần sửa lại content type của request từ ```application/x-php``` sang ```image/jpeg``` rối send lại ![image](https://github.com/Myozz/Web_Applications/assets/94811005/d4d3e9f1-d555-46d0-a14e-dbb7ae2fbda2) và upload thành công :||
  - Có thể quay lại panel để mở img ấy ra :|| hoặc send cái ```/file/avatars/hack.php``` thì sẽ chả ra content của file secret
