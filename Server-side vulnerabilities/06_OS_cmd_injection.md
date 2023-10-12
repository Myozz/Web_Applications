# Nói qua về các injection
- Tôi sẽ để nguyên là injection chứ không dịch ra trong suốt quá trình viết cái này
- Nói qua về injection đi, cứ tưởng tượng web nó có một cái form để điền vào, cái box điền được cấu tạo bằng code kiếc thứ, việc nhập dữ liệu vào có nghĩa là bạn đang chèn dữ liệu vào dòng code đấy để khi submit thì nó sẽ thực thi
- Nếu đủ hiểu biết thì ta hoàn toàn có thể lợi dụng việc này để đoàn vào box các kí tự nhằm khiến code chạy theo ý của bản thân
- Có đủ kiểu injection (SQL Injection là cái tôi thấy nhiều nhất). Còn ở đây chúng ta sẽ nghiên cứu về OS cmd injection, nghe tên thôi là biết nó sẽ sử dụng các câu lệnh của hđh rồi

# OS CMD Injection là gì?
- OS cmd injection hay còn được biết đến là shell injection. Nó cho phép atker thực thi OS cmd trên server đang chạy app, mà đương nhiên thường thì là để làm tổn hại tới app và dữ liệu của nó. Thường thì, một atker có thể tận dụng OS cmd injection để làm tổn hại tới các phần khác của cơ sở hạ tầng, và khai thác mối quan hệ tin cậy để atk vào các hệ thống khác bên trong tổ chức

# Các câu lệnh hữu dụng
- Sau khi bạn xác định được OS cmd injection, khá ngon để thực thi vài con lệnh mở đầu để thu thập thông tin về hệ thống. Dưới đấy là một số cmd hữu dụng trên Linux và Windows:

![image](https://github.com/Myozz/Web_Applications/assets/94811005/857b5c62-9237-4ad9-81d2-ed8e6bbc8506)
- Thực tế thì tôi dùng mỗi whoami :(

# Injecting OS cmd
- Trong ví dụ này, một app shopping cho user xem một mặt hàng có còn hàng trong một cửa hàng cụ thể không. Thông tin này được truy cập thông qua một URL

      https://insecure-website.com/stockStatus?productID=381&storeID=29
- Để cung cấp thông tin hàng, app sẽ phải truy vấn hệ thống. Tính năng được thực hiện bằng cách gọi ra một shel command với sản phẩm và lưu trữ IDs như đối số

      stockreport.pi 381 29
- Câu lệnh này cho ra trạng thái hàng của sản phẩm cụ thể, rồi tống lại cho user

# Injection OS cmd - tiếp
- Khi một app không có một sự phòng vệ tốt trước OS cmd Injection, một atker có thể gửi input dưới đây để thực thi một câu lênh bất kì

      & echo aiwèwiguh &
- Nếu input này được gửi trong ```productID```, command được thực thi sẽ là

      stockreport.pi & echo aiwèwiguh & 29
- Command ```echo``` có thể khiến cho chuỗi tham số của nó lặp lại ở đầu ra (vd echo myozz sẽ trả về myozz). Đây là một cách hữu dụng để test cho một số loại OS cmd Injection. ```&``` dùng để phân cách các cmd liên tiếp. Trong ví dụ này, sẽ có 3 cmd được thực thi theo thứ thự. Output trả về sẽ như sau:

      Error - productID was not provided
      aiwefwlguh
      29: command not found
  - Ba dòng output này chứng minh rằng:
    - ```stockreport.pi``` vẫn sẽ được thực thi mà không cần đối số, và nó trả về một thông báo lỗi
    - Lệnh ```echo``` được thêm vào cũng được thực thi đúng với những gì được điền vào
    - ```29``` ở cuối cũng được thực thi như một cmd và báo lỗi
- Việc đặt ```&``` giữa các cmd rất hữu dụng bởi nó khiến cmd được inject vào thực thi riêng biệt. Giảm khả năng bị ngăn chặn thực thi
## Lab lú lẫn
- Lab này chứa một OS cmd injection trong stock checker
- App sẽ thực thi một shell cmd gồm user-supplied product và store IDs, rồi trả lại raw output từ cmd
- Để giảilab này, chỉ cần thực thi được ```whoami``` là xong
- Lab này chắc sẽ dễ thôi :||
- View detailts một cái item bất kì ![image](https://github.com/Myozz/Web_Applications/assets/94811005/f48e8671-8564-45d3-b7ab-a1256300c31c)
- Bật intercept lên và check stock thử ![image](https://github.com/Myozz/Web_Applications/assets/94811005/2d136713-84ca-490b-8d1f-1d60098cfe55)
- Dòng cuối cùng sẽ là một nơi lí tưởng để thực thi os cmd injection, tôi sẽ send nó sang repeater để test chuẩn chỉ hơn
- ```productId=1&storeId=1``` thì tức là shell cmd sẽ bili ```stock.pi 1 1```
  - Vậy thì chỉ cần thêm một đoạn lệnh ở request là được, tôi sẽ thêm ```|whoami``` vào cuối, tôi đã thử với & nhưng nó chỉ trả về 62 thôi và nó cũng không cho bỏ tham số trong storeID và productID
  - Thêm vào thì send là xong rồi :||
