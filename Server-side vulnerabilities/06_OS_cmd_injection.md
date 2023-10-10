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
