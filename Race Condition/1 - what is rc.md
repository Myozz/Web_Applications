# Race conditions (Điều kiện thực hiện)
- Là một loại lỗ hổng phổ biến liên quan đến lỗi logic kinh doanh. Chúng xảy ra khi web thực hiện request kiểm nhiểm mà không có những biện pháp bảo mật đầy đủ. Điều này có thể dẫn tới các mối đe doạ tương tác với cùng một dữ liệu tại cùng một thời điểm, hậu quả là sự "va chạm" (collision) mà có thể gây ra những hành vi ngoài ý muốn và có thể bị khai thác cho mục đích xấu
- Khoảng thời gian xảy ra va chạm còn được gọi là **race window** (cửa sổ đua). Khoảng này có thể là một phần bao nhiêu đó của một giây giữa 2 tương tác với dtb.
- **Ví dụ**: Như nhũng lỗi logic khác, tác đọng của một race condition phụ thuộc nhiều vào app và tính năng nhất định mà tại đó nó xảy ra
- Ở phần này, ta sẽ nghiên cứu làm sao để xác định và khai thác các loại khác nhau của race condition. Ta sẽ xem cách mà Burp Tool hỗ trợ ta vượt qua thử thách để thực hiện atk cơ bản, cộng thêm một phương pháp đã được kiểm/thử nghiệm mà cho phép bạn phát hiện được tình trạng của race condition bên trong các tiến nhưng đa bước ẩn (hidden multi-step). Những thứ này vượt qua giới hạn mà bạn có thể quen thuộc?

# Limit overrun race condition (Điều kiện thực hiện vượt qua giới hạn)
- Loại race condition được biết tới nhiều nhất cho phép bạn vượt qua một số loại giới hạn được sử dụng bởi bussiness logic của app
- Ví dụ, hãy xét một cửa hàng online cho phép bạn nhập một promo code để có được phiếu giảm giá. Để áp dụng phiếu giảm giá đó, ap sẽ thực hiện những tác vụ bậc cao:





