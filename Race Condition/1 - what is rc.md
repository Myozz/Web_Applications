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
- Như bạn có thể thấy, app chuyển sang trạng thái phụ tạm thời. Trạng thái mà đi vào rồi exit trước khi request đang chạy hoàn thành. Trong trường hợp này, trạng thái phụ bắt đầu khi server bắt đầu thực hiện request đầu tiền, và kết thúc khi nó cập nhật dữ liệu trong dtb để xác định xem ta sử dụng code hay chưa. Điều này giới thiệu một cửa sổ đua (race window) nhỏ trong suốt thời điểm mà bạn có thể dùng code bao nhiêu lần tuỳ ý




